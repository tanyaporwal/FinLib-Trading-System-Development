How to provide an additional TimeDataSeries within Strategy usage

The Function AddTimeDataSeries is used in your strategy initialization method to get a handle of custom TimeDataSeries. This method must be implemented by your program through a interface ITimeDataSeriesAccessor

**Assign your implementation class to your strategy using a method TimeDataSeriesAccessor**
```
// this return highest value of close since the second most recent occurrence of the high being above 50

public TimeDataSeries AddTimeDataSeries(ExchangeSegment exchangeSegment, 
                                                                              string scripIdentifier, 
                                                                              DateTime fromDateTime,
                                                                              int compressionMultiplier = 1)


// Implement your Fill TimeDataSeries bars through csv of DB source here
public class TimeDataSeriesAccessorImpl : ITimeDataSeriesAccessor
{
        public TimeDataSeries GetTimeDataSeries(ExchangeSegment exchangeSegment,
                                                                                        string instrumentIdentifer, DateTime fromDateTime, int compressionMultiplier)
        {
            TimeDataSeries timeDataSeries = new TimeDataSeries(BarCompression.MinuteBar,
                                                                   60 * compressionMultiplier,
                                                                   BarType.CandleStick,
                                                                   new TimeSpan(9, 15, 0),     // Session Start time
                                                                   new TimeSpan(15, 30, 0));   // Session End time);

           // Write a code here to fill a TimeDataSeries bars through csv or DB source here

            return timeDataSeries;
        }
 }
```
// Backtesting code

ITimeDataSeriesAccessor timeDataSeriesAccessorImpl = new TimeDataSeriesAccessorImpl();
strategyBase.TimeDataSeriesAccessor = timeDataSeriesAccessorImpl;
…………………
…………………
TSExecutor tsExecutor = new TSExecutor(strategyConfigData, instrument, timeDataSeries, strategyBase);
tsExecutor.Run();


