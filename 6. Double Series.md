How to store calculation in DoubleSeries?

There can be need to store some calculated value during a ProcessStrategy for further decision evaluation. User mostly use Indicator for same purpose but for simple calculation DoubleSeries is also recommended datastructure to store bar index based calculated values.

For example following code snippet explain on how to store a range of bar that can be further used for some decision logic
private DoubleSeries _barRangeDoubleSeries = new DoubleSeries();
……
……

public override void ProcessStrategy()
{
    _barRangeDoubleSeries[CurrentIndex] = High(CurrentIndex) – Low(CurrentIndex); 

   if(barRangeDoubleSeries[CurrentIndex] > barRangeDoubleSeries[CurrentIndex-1])
   {
       EnterLongAtMarket(1);
   }

   double aveareRangeBar = _barRangeDoubleSeries.Average(10);
}


Note here DoubleSeries also have most of the methods and properties like Average, Standard Deviation as exist for StrategyBase

BarsSince function usage

There can be instance where you search for specific bar based on its index where user defined condition is satisfied. The evaluation return the bar index where Nth occurrence of condition is satisfied.

In following example user declares a Func<T,TResult> variable and assigns it a lambda expression that evaluates of close price is greater than open price. The Function return type must be bool and it iterates through bar index from most recent one till the lambda expression return true of Nth occurrence
Func<int, bool> Evaluate = (barIndex) => tdSeries[barIndex].Close > tdSeries[barIndex].Open;

// check only first occurance of the condition
int barIndexMatched = BarsSince(Evaluate, 1);
double closePriceOfMatchingBar = Close(barIndexMatched);

ValueWhen function usage

There can be instance where you search for specific bar value where user defined condition is satisfied. The evaluation return the bar value based on user defined price type where Nth occurrence of condition is satisfied.

In following example user declares a Func<T,TResult> variable and assigns it a lambda expression that evaluates all bars till the bar time is not equal to 09:16 AM and return the close price of the matching bar. The Function return type must be bool and it iterates through bar index from most recent one till the lambda expression return true of Nth occurrence
Func<int, bool> Evaluate = (barIndex) => tdSeries.TimeNum(barIndex) == 091600;
double closePrice = tdSeries.ValueWhen(Evaluate, 'C', 1);


HighestSince function usage

Return the Highest value of price value array from most recent bar till user defined criteria is met
// this return highest value of close since the second most recent occurrence of the high being above 50
Func<int, bool> Evaluate = (barIndex) => High(barIndex) > 50;
double highestVal = HighestSince(Evaluate, 'C', 2);

LowestSince function usage

Return the Lowest value of price value array from most recent bar till user defined criteria is met
// this return highest value of close since the second most recent occurrence of the high being above 50
Func<int, bool> Evaluate = (barIndex) => High(barIndex) > 50;
double lowestVal = LowestSince(Evaluate, 'C', 2);
