### Creating Your First Trading Strategies

A strategy is a collection of logic or rules that may use many indicators or price actions to determine when to buy and when to sell some instruments. A strategy can typically be run against historical data (backtested) or run against live data to execute real-time trades. There are many indicators packaged with QX.FinLib and any proprietary custom indicators can be easily build. We’ll be walking through Strategy creation which could theoretically use any Indicator to perform buys and sells.


The building block of any basic trading strategies used to analyze and backtest are:

**Historical Data:** The historical data of given financial asset is mainly represented the past price events in the form of OHLCV bar of specific  time frame compression  i.e. 5 minutes, 1 hours etc. The historical data exist with you in either csv format or fetched from data provider using their APIs or stored in any database repository. 

This data must be represented or converted to TimeDataSeries object
The asset class is represented by object InstrumentBase of types Equity, Futures, Options etc

**The strategy model:** The strategy model is a rule based system that generates signals for entering and exit from trade. This is represented by your own .NET based class that must inherit from QX.FinLib.TS.Strategy.StrategyBase and implements some override method and logics specific to signal generation.

**The strategy Configuration or Settings:** The backtesting process of your strategy needs some setup to define for example initial capital, any commission or adjustment of Slippages per trade etc.

Following are example of setting some strategy configuration details

StrategyConfigData strategyConfigData = new StrategyConfigData();
strategyConfigData.InitialCapital = 500000;

strategyConfigData.Pyramiding = PyramidingType.Disallow;
strategyConfigData.MaxPyramidingPosition = 1;

strategyConfigData.ContractMultiplier = 1;
strategyConfigData.StopLossTriggerPoint = StopLossTriggerPoint.High_Low_Bar;
strategyConfigData.OrderExecutionAt = OrderExecution.CurrentBar_Close;

Following are the StrategyConfigData properties:
| | |
| :--- | :--- |
| InitialCapital |  |
| FilledOrderCommission |  |
| CancelOrderCommission |  |
| LimitOrderSlippage |  |
| MarketOrderSlippage |  |	
| CommissionType | Per_Contract, Per_Transaction, Percentage |
| StopLossTriggerPoint |  |
| SlippageType | Per_Contract, Per_Transaction, Percentage |
| SlippagePercentage |  |
| OrderExecutionAt | NextBar_Open, CurrentBar_Close |
| Pyramiding |   |
| MaxPyramidingPosition	|   |
| ContractMultiplier |   |
| UserID |  |
| StrategyName |   |
| PositionSizerType	| FixedSize, InitialCapitalSize, FixedDollar, TradePercentageOfAccountBalance, RiskPercentageOfAccountBalace, Custom |
| PositionSize	|   |

**Following framework DLL references are needed for strategy development code:**

using QX.FinLib.Data;
using QX.FinLib.Data.TI;
using QX.FinLib.TS.Strategy; 

Dll libraries are added by clicking the References button in the Source code tab.

Then you create your class named with Strategy and inherit it from StrategyBase class

All trading strategy class has following basic template
```

using QX.FinLib.Data;
using QX.FinLib.Data.TI;

using QX.FinLib.TS.Strategy;

namespace QX.Strategies
{
    [StrategyAttribute("{B9650BDC-74EB-451E-A05E-2BF7056A2BFA}", "EMAStrategy", "Exponential Moving Average Strategy", "QXT")]
    public class EMAStrategy : StrategyBase
    {
        [Parameter("FastLengthPeriod", Description = "The fast length of the EMA", DefaultValue = 12)]
        private int _fastLengthPeriod = 5;

        [Parameter("SlowLengthPeriod", Description = "The slow length of the EMA", DefaultValue = 15)]
        private int _slowLengthPeriod = 12;

        private IndicatorBase _slowEMAIndicator;
        private IndicatorBase _fastEMAIndicator;

        public EMAStrategy()
        {

        }
        protected override void Initialize()
        {
            // Initialize the indicator object
            _slowEMAIndicator = GetIndicator("EMA);
            _fastEMAIndicator = GetIndicator ("EMA");
        }

        protected override void OnStateChanged()
        {
            if (StrategyState == StrategyState.ActiveMode)
            {
               // Change the indicator input based on user specified input signal settings
                _slowEMAIndicator.SetFieldValue("LengthPeriod", _slowLengthPeriod);
                _fastEMAIndicator.SetFieldValue("LengthPeriod", _fastLengthPeriod);
            }
        }

        public override void ProcessStrategy()
        {
            int contractSize = 1;

            if (CurrentIndex < _slowLengthPeriod)
                return;

            CrossOver crossover = CrossOver.None;
            crossover = _fastEMAIndicator["EMA"].Cross(_slowEMAIndicator["EMA", CurrentIndex]);

            if (crossover == CrossOver.Above)
            {
                EnterLongAtMarket(contractSize, "EnterLong");
            }
            else if (crossover == CrossOver.Below)
            {
                EnterShortAtMarket(contractSize, "EnterShort");
            }            
        }

        public override void OnMarketData(IMarketDepth marketDepth)
        {
            // For real time events
        }

        public override void OnPositionChange(int marketPosition)
        {

        }
    }
}
```
The Strategy class must be inherited from a framework provided StrategyBase class and it provides various override method 

### Strategy Input parameters 

**Following is code snippet to define a input parameter for your strategy:**

[Parameter("FastLengthPeriod", Description = "The fast length of the EMA", DefaultValue = 12)]
private int _fastLengthPeriod = 5;

There can be any number of inputs define for strategy and used to change the state of strategy instance while running a backtesting. Any variable decorated by attribute Parameter is tagged as input parameter by system

As compared to Indicator template there is no concept of Output parameter and only order signal generated from strategy is the main return type during incremental backtesting process from a method ProcessStrategy

#### Initialze  method 

The initialize method is called once prior to running the strategy and used to set properties and initialize indicator that are used during backtesting evaluation process.
Following code snippet initialize the two EMA Indicator instances
 protected override void Initialize()
  {
            // Initialize the indicator object
            _slowEMAIndicator = GetIndicator("EMA);
            _fastEMAIndicator = GetIndicator ("EMA");
 }

The GetIndicator creates instance of EMA indicator with default main TimeDataSeries used in backtesting.

#### OnStateChanged  method

The OnStateChanged method is called with change in state of strategy instance. The strategy inastance is undergoing various state change and OnStateChanged method allows to initialize and reset some variables depending on the logic.
protected override void OnStateChanged()
{
            if (StrategyState == StrategyState.ActiveMode)
            {
                // Change the indicator input based on user specified input signal settings
                _slowEMAIndicator.SetFieldValue("LengthPeriod", _slowLengthPeriod);
                 _fastEMAIndicator.SetFieldValue("LengthPeriod", _fastLengthPeriod);
            }
}


**Following are the different state:**
|  |  |
| :--- | :--- |
| ConfigureMode	| This is initial state before a backtesting starts. |
| ActiveMode | This is state just before a backtesting starts. Here we usually set any indicator user specified input value. |
| HistoricalMode | This is state if process is undergoing a backtesting on historical data. |
| RealtimeMode | This is state after backtesting, in case system is attached with realtime data. |
| TerminateMode | This is state when strategy instance is terminating. |

There is important properties that is useful in case of system is setup for realtime mode and is called CalculationMode 
It has two options:
|  |  |
| :--- | :--- |
| OnBarClose | The method ProcessStrategy is called only on Bar close event and it is a default behaviour. |
| OnEachTick | The method ProcessStrategy is called on each LTP change which may change the High, Low, Close of a current developing bar. The indicator value change during a developing bar process. This mode is only valid when trading system is attached to realtime data stream otherwise system behave like OnBarClose mode as default or fallback. |

```
protected override void OnStateChanged()
{
            if (StrategyState == StrategyState.ConfigureMode)
            {
                CalculationMode = CalculationMode.OnEachTick;
            }
}
```

### ProcessStrategy  method 

This function is entry point of your strategy logic and here we put our strategy main decision logic.
ProcessStrategy method is called for every bar close or for each incoming tick depending on setup is for backtetsing or realtime or dual mode. 

For historical bar defined by TimeDataSeries, this method is called at least once for each bar and here programmers can evaluate a signal generation logic based on indicator or price actions.
The signal is order action and invoke by command like EnterLong, EnterShort, GoFlat etc.



#### Various action command supported by strategy:
**EnterLongAtMarket**
Generates a buy market order to create a long position.

If current position is 0, then action 
**EnterLongAtMarket(10, "long signal due of sma crossover")**
will lead to long position of 10.

If current position is 10, then action 
**EnterLongAtMarket(10, "long signal due of sma crossover")**
will ignore the command as position is already long. There is exception to this rule if through strategy configuration we have allowed Pyramidding ON,then this leads to a position of 20.

if current position is -12 short, then action 
**EnterLongAtMarket(10, "long signal due of sma crossover")**
will lead to long position of 10, which have generated due to Buy action of 20 quantity.

The filled price is either close of current bar or open of next bar depending on strategy configuration setting 

StrategyConfigData strategyConfigData = new StrategyConfigData();
strategyConfigData.OrderExecutionAt = OrderExecution.CurrentBar_Close;

strategyConfigData.OrderExecutionAt = OrderExecution.NextBar_Open;

**EnterShortAtMarket**
Generates a sell market order to create a short position.

If current position is 0, then action 
**EnteShortAtMarket(10, "short signal due of sma crossover")**
will lead to short position of 10.

In next bar iteration the call of MarketPosition API in following way 
if(MarketPosition == -10) will return true


**EnterLongAtPrice(int quantity, double price, string shortInfo)**
Generates a long position with a specific user defined filled price. This is different than a limit and the order is immediately filled on user specified price if it fall between bar high and low, otherwise it fill exactly in same manner as EnterLongAtMarket.

**EnterShortAtPrice(int quantity, double price, string shortInfo)**
Generates a short position with a specific user defined filled price. 


**EnterLongAtLimit(int quantity, double limitPrice, string shortInfo)**
Generates a long position when user specified limit price condition is reached.
**EnterShortAtLimit(int quantity, double limitPrice, string shortInfo)**
Generates a short position when user specified limit price condition is reached.
**EnterLongAtStop(int quantity, double limitPrice, string shortInfo)**
Generates a long position when user specified stopmarket  price condition is reached.
**EnterShortAtStop(int quantity, double limitPrice, string shortInfo)**
Generates a short position when user specified stop market price condition is reached.
**EnterLongAtStopLimit(int quantity, double stopPrice, double limitPrice, string shortInfo)**
Generates a long position when a stop limit price is trigger
**EnterShortAtStopLimit(int quantity, double stopPrice, double limitPrice, string shortInfo)**
Generates a short position when a stop limit price is trigger
**GoFlatAtMarket(string shortInfo)**
Squareoff any long or short position at market price

**GoFlatAtLimit(double limitPrice, string shortInfo)**
Squareoff any long or short position at limit price

**GoFlatAtPrice(double orderPrice, string shortInfo)**
Squareoff any long or short position at user specified price. In live scenario this may be treated as market price

**Important properties and function mainly used in ProcessStrategy()**
Int CurrentIndex	Return the current bar Index, the first bar index is 0
IBarData TimeDataSeries[int]	Return the Bar at specific index
IBarData barData = TimeDataSeries[CurrentIndex];
if(barData.Close > barData.Open)
{
}
If user trying to go beyond CurrentIndex or negative indexes it return null;
double BarValue(char priceType, int barIndex)	Return the value at specific barIndex for user specified price type
‘O’ – Open Price, ‘H’ – High Price, ‘L’ – Low Price, ‘C’ – Close Price
// return a close price 1 bar ago
double closePrice = BarValue('C', CurrentIndex - 1);
double Ref(char priceType = 'C', int refBarIndex = 0)	Return the price value relative to current bar index.
// return a close price 1 bar ago, for all previus bar ref index must be negative
double closePrice = Ref('C', -1);
// recent close price
double closePrice = Ref('C', 0);
double RefX(char priceType = 'C', int refBarIndex = 0)	Return the price value relative to current bar index.
// return a close price 1 bar ago, the ref Index must be positive
double closePrice = RefX('C', 1);
int TimeNum()	Return the bar time in format HHmmss
If bar index time is 09:35:00, it return 093500. 
Note here the bar time always point towards the start of the bar
If logic need to be placed to take entry intraday position between two times
int currentTime = TimeNum();
if (currentTime >= TradeStartTime && currentTime < TradeStartTime)
{
}
int DateNum()	Return the bar date in format yyyyMMdd
If bar index date is 28-01-2022, it return 220128. 

int MarketPosition	Return the current market position
A value 0 indicate no position
A value greater than 0 indicate the long position
A value less than 0 indicate the short position
double GetAverageEntryTradedPrice()	In case the system has position, it return the average trade price of recent entry trades
double GetLastTradeEntryPrice()	In case the system has position, it return the trade price of last open trade
bool IsFirstBarOfDay()	Indicate if the current bar is the first bar of the day
bool IsLastBarOfDay()	Indicate if the current bar is the last bar of the day
int DayOfWeekNum()	Return the week num of current bar.1-Monday, 2-Tuesday,….0-Sunday
int DayOfMonthNum()	Return the month num of the current bar. 1-Jan, 2-Feb,….12-Dec
bool IsFirstDayOfWeek()	If the current bar is for Monday then it returns true
IBarData GetDailyTimeFrameBar(int refBarIndex = 0, bool includeTodaysDate = true)	Return the daily bar based on refBarIndex
GetDailyTimeFrameBar(-1) return the previous day OHLC bar from current lower bar compression of minutes bar.
double LLV(int lengthPeriod)
double LLV(char priceType, int lengthPeriod)
double LLV(char priceType, int lengthPeriod, int barIndex)
	Return the lowest low bar value of price type for N length period from most recent bar

if barIndex is specified it return Lowest low bar value starting from barIndex till N length period ago bar

// return 10 period lowest low value of price type Low of bar
double llv = LLV('L',  10);
double HHV(int lengthPeriod)
double HHV(char priceType, int lengthPeriod)
double HHV(char priceType, int lengthPeriod, int barIndex)
	Return the highest high bar value of N length period from most recent bar

double StdDev(int lengthPeriod)
double StdDev(char priceType, int lengthPeriod, int barIndex)
	Return the standard deviation 
double Sum(int lengthPeriod)
double Sum(char priceType, int lengthPeriod, int barIndex)
	Return the sum of N period bar 
double Average(int lengthPeriod)
double Average (char priceType, int lengthPeriod)
double Average (char priceType, int lengthPeriod, int barIndex)	Return the average of N period bar
double WeightedAverage(int lengthPeriod)
double WeightedAverage(char priceType, int lengthPeriod, int barIndex)	Return the weighted average of bar close or user specified price type for N period
double EMAValue(double presentValue, double previousValue, int lengthPeriod)	Return the exponential moving average value for N length period
double GetTrueLow(int index)	
double GetTrueHigh(int index)	
double TrueRange(int barIndex)	
double TypicalPrice(int barIndex)	
double CLV(int index)	
bool IsGapUp()	
bool IsGapDown()	
bool IsInside()	
bool IsRising(int index)	
bool IsFalling(int barIndex)	
double UpperShadow(int barIndex)	
double LowerShadow(int barIndex)	
int BarsSince(
    Func<int, bool> evaluateFuncAtBarIndex,
     int nthOccuranceTimes = 1)	The BarsSince return a bar index when user defined condition becomes true from the latest bar
double HighestSince(
Func<int, bool> evaluateFuncAtBarIndex, 
char priceType = 'C', int nthOccuranceTimes = 1)	The HighestSince return a high value of price type in the array of all data points till user defined condition is true from the latest bar
public double LowestSince(
    Func<int, bool> evaluateFuncAtBarIndex,
    char priceType = 'C', int nthOccuranceTimes = 1)	The LowestSince return a low value of price type in the array of all data points till user defined condition is true from the latest bar
public double ValueWhen(
    Func<int, bool> evaluateFuncAtBarIndex, 
    char priceType = 'C', int nthOccuranceTimes = 1)	The ValueWhen return a value of price type at bar index when user defined condition is true from the latest bar
IBarData Lookup(DateTime dateTime)	Return a prior BarData specific to user defined datetime index
IBarData Lookup(int timeNum)	Return a prior BarData specific to user defined timenum index
CrossOver Cross(DoubleSeries firstSeries, DoubleSeries secondSeries)	Indicate CrossOver.Above, CrossOver.Below or None based on two series

> Note: that most of the function is also applied to DoubleSeries and your indicator handles used in your strategy class



