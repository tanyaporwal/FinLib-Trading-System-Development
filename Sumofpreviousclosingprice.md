How to get Sum of previous N bar closing price in ProcessStrategy  method
// Method 1
public override void ProcessStrategy()
{
    // Do not proceed if we are not having enough bar
    if (Input_LengthPeriod < CurrentIndex)
        return;

    double sum = 0;
    for (int iCnt = CurrentIndex; iCnt > CurrentIndex - Input_LengthPeriod; iCnt--)
    {
        sum += Close(iCnt);
    }
}


// Method 2
double sum = 0;
for (int barsAgo = 0; barsAgo < Input_LengthPeriod; barsAgo++)
{
    sum += RefX('C', barsAgo);
}

// Method 3
double sum = Sum('C', Input_LengthPeriod);


The sum value return is NaN till BarIndex is less than Input_LengthPeriod if you are using predefined library functions
