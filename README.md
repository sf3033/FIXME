# FIXME
//FIXME
 think I find a systematic bug in pandas.DataFrame in python.

I was trying to build an backtest class for my trading strategy. I have two DataFrames, one stores all the stock pair I am currently holding called HoldingPair, the other named TradingHis storing all the transaction history.

Here is the thing, I checked HoldingPair everyday to decide to I should sell the current holding, and if I do, I delete the pair from my HoldingPair DataFrame. In the meanwhile, I search the price data to see if there is anything I should buy, and if I do I add the trade into HoldingPair.

One strange thing is while the program running perfectly well in the beging 4 years, at a specific point, I cannot access the HoldingPair DataFrame I built, if I ever wanted to do anything with this DataFrame, the program crashed. Here comes the code(I print HoldingPair DataFrame every time I add or drop anything in it.)

This is where I insert a pair into the dataframe:

        if spread>=Spreadmean+Std:
            symbol=datetime.now()
            Holding1={'Ticker':pair[0],'Position':-invest/PriceData.loc[date][pair[0]],'BuyingPrice':PriceData.loc[date][pair[0]],'BuyingDate':date,'SellingPrice':0,'SellingDate':0,'Symbol':symbol}
            Holding2={'Ticker':pair[1],'Position':invest/PriceData.loc[date][pair[1]],'BuyingPrice':PriceData.loc[date][pair[1]],'BuyingDate':date,'SellingPrice':0,'SellingDate':0,'Symbol':symbol}
            insert={'Pair':pair,'IfShortFirst':1,'Criterion':Std,'SpreadMean':Spreadmean,'Symbol':symbol}
            print(insert)
            print(HoldingPair)
            print('before inserting')
            HoldingPair.loc[HoldingPair.shape[0]]=insert
            print(HoldingPair)
            print("insert done")
            TradingHis.loc[TradingHis.shape[0]]=Holding1
            TradingHis.loc[TradingHis.shape[0]]=Holding2
            #print('holding insert done')
        elif spread<=Spreadmean-Std:
            symbol=datetime.now()
            Holding1={'Ticker':pair[0],'Position':invest/PriceData.loc[date][pair[0]],'BuyingPrice':PriceData.loc[date][pair[0]],'BuyingDate':date,'SellingPrice':0,'SellingDate':0,'Symbol':symbol}
            Holding2={'Ticker':pair[1],'Position':-invest/PriceData.loc[date][pair[1]],'BuyingPrice':PriceData.loc[date][pair[1]],'BuyingDate':date,'SellingPrice':0,'SellingDate':0,'Symbol':symbol}
            insert={'Pair':pair,'IfShortFirst':-1,'Criterion':Std,'SpreadMean':Spreadmean,'Symbol':symbol}
            insert={'Pair':pair,'IfShortFirst':1,'Criterion':Std,'SpreadMean':Spreadmean,'Symbol':symbol}
            print(insert)
            HoldingPair.loc[HoldingPair.shape[0]]=insert
            print(HoldingPair)
            print("insert done")
            TradingHis.loc[TradingHis.shape[0]]=Holding1
            TradingHis.loc[TradingHis.shape[0]]=Holding2 
This is where I drop I a row when selling:

def CloseIdea(date):
global HoldingPair
for indexs in HoldingPair.index:
    if HoldingPair.loc[indexs]['IfShortFirst']==1:
        spread=PriceData.loc[date,HoldingPair.loc[indexs,'Pair'][0]]-PriceData.loc[date,HoldingPair.loc[indexs,'Pair'][1]]
        if spread<=HoldingPair.loc[indexs]['SpreadMean']:
            print("Dropping:")
            print(HoldingPair.loc[indexs])
            index1=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][0]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][0]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index1,'SellingPrice']=PriceData.loc[date][HoldingPair.loc[indexs]['Pair'][0]]
            index2=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][0]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][0]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index2,'SellingDate']=date
            index1=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][1]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][1]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index1,'SellingPrice']=PriceData.loc[date][HoldingPair.loc[indexs]['Pair'][1]]
            index2=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][1]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][1]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index2,'SellingDate']=date
            HoldingPair=HoldingPair.drop(indexs)
            print('after dropping')
            print(HoldingPair)
    else:
        spread=PriceData.loc[date,HoldingPair.loc[indexs]['Pair'][0]]-PriceData.loc[date][HoldingPair.loc[indexs]['Pair'][1]]
        if spread>=HoldingPair.loc[indexs]['SpreadMean']:
            print("Dropping:")
            print(HoldingPair.loc[indexs])
            index1=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs,'Pair'][0]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][0]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index1,'SellingPrice']=PriceData.loc[date,HoldingPair.loc[indexs,'Pair'][0]]
            index2=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs,'Pair'][0]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][0]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index2,'SellingDate']=date
            index1=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs,'Pair'][1]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][1]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index1,'SellingPrice']=PriceData.loc[date,HoldingPair.loc[indexs,'Pair'][1]]
            index2=TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs,'Pair'][1]].index].loc[TradingHis.loc[TradingHis[TradingHis['Ticker']==HoldingPair.loc[indexs]['Pair'][1]].index]['Symbol']==HoldingPair.loc[indexs]['Symbol']].index[0]
            TradingHis.loc[index2,'SellingDate']=date
            HoldingPair=HoldingPair.drop(indexs) 
            print('after dropping')
            print(HoldingPair)

    print("close idea done")
Now I am showing you the unbelievable crash:

    2007-01-05 00:00:00
    {'Pair': ['HP', 'EQT'], 'IfShortFirst': 1, 'Criterion': 
    0.30538302270672896, 'SpreadMean': -16.923524363882645, 'Symbol': 
    datetime.datetime(2018, 3, 6, 14, 6, 31, 130638)}
    Empty DataFrame
    Columns: [Criterion, IfShortFirst, Pair, SpreadMean, Symbol]
    Index: []
    before inserting
    Criterion  IfShortFirst       Pair  SpreadMean                     
    Symbol
    0   0.305383           1.0  [HP, EQT]  -16.923524 2018-03-06 
    14:06:31.130638
    insert done
    {'Pair': ['MRO', 'HAL'], 'IfShortFirst': 1, 'Criterion': 
    0.3881482591437784, 'SpreadMean': -5.4640946690708985, 'Symbol': 
    datetime.datetime(2018, 3, 6, 14, 6, 31, 143521)}
    Criterion  IfShortFirst       Pair  SpreadMean                     
    Symbol
    0   0.305383           1.0  [HP, EQT]  -16.923524 2018-03-06 
    14:06:31.130638
    before inserting
      Criterion  IfShortFirst        Pair  SpreadMean                     
    Symbol
    0   0.305383           1.0   [HP, EQT]  -16.923524 2018-03-06 
    14:06:31.130638
    1   0.388148           1.0  [MRO, HAL]   -5.464095 2018-03-06 
    14:06:31.143521
    insert done
    close idea done
    close idea done
    2007-01-08 00:00:00
    Dropping:
    Criterion                         0.305383
    IfShortFirst                             1
    Pair                             [HP, EQT]
    SpreadMean                        -16.9235
    Symbol          2018-03-06 14:06:31.130638
    Name: 0, dtype: object
    after dropping
       Criterion  IfShortFirst        Pair  SpreadMean                     
    Symbol
    1   0.388148           1.0  [MRO, HAL]   -5.464095 2018-03-06 
    14:06:31.143521
    close idea done
    close idea done
    2007-01-09 00:00:00
    close idea done
    2007-01-10 00:00:00
    close idea done
    2007-01-11 00:00:00
    close idea done
    2007-01-12 00:00:00
    close idea done
    2007-01-16 00:00:00
    close idea done
    2007-01-17 00:00:00
    close idea done
    2007-01-18 00:00:00
    close idea done
    2007-01-19 00:00:00
    close idea done
    2007-01-22 00:00:00
    close idea done
    2007-01-23 00:00:00
    close idea done
    2007-01-24 00:00:00
    close idea done
    2007-01-25 00:00:00
    close idea done
    2007-01-26 00:00:00
    close idea done
    2007-01-29 00:00:00
    close idea done
    2007-01-30 00:00:00
    close idea done
    2007-01-31 00:00:00
    close idea done
    2007-02-01 00:00:00
    close idea done
    2007-02-02 00:00:00
    close idea done
    2007-02-05 00:00:00
    {'Pair': ['OKE', 'EOG'], 'IfShortFirst': 1, 'Criterion': 
    1.2238298117734636, 'SpreadMean': -18.4578382164355, 'Symbol': 
    datetime.datetime(2018, 3, 6, 14, 6, 33, 176755)}
    Traceback (most recent call last):

      File "<ipython-input-36-2e3dbdb2cca1>", line 1, in <module>
        runfile('/Users/sikaifeng/Desktop/文件/学习/Statistics 
    Albitrage/homework/project/TradingQ.py', wdir='/Users/sikaifeng/Desktop/文件/学习/Statistics 
    Albitrage/homework/project')

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-
    packages/spyder/utils/site/sitecustomize.py", line 705, in runfile
        execfile(filename, namespace)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-
    packages/spyder/utils/site/sitecustomize.py", line 102, in execfile
        exec(compile(f.read(), filename, 'exec'), namespace)

      File "/Users/sikaifeng/Desktop/文件/学习/Statistics 
    Albitrage/homework/project/TradingQ.py", line 152, in <module>
        ExecuteStrategy()

      File "/Users/sikaifeng/Desktop/文件/学习/Statistics 
    Albitrage/homework/project/TradingQ.py", line 147, in ExecuteStrategy
        OpenIdea(indexs)

      File "/Users/sikaifeng/Desktop/文件/学习/Statistics 
    Albitrage/homework/project/TradingQ.py", line 78, in OpenIdea
        HoldingPair.loc[HoldingPair.shape[0]]=insert

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/indexing.py", line 179, in __setitem__
self._setitem_with_indexer(indexer, value)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/indexing.py", line 583, in _setitem_with_indexer
setter(item, v)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/indexing.py", line 513, in setter
s._data = s._data.setitem(indexer=pi, value=v)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/internals.py", line 3203, in setitem
return self.apply('setitem', **kwargs)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/internals.py", line 3091, in apply
applied = getattr(b, f)(**kwargs)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/internals.py", line 686, in setitem
values, _, value, _ = self._try_coerce_args(self.values, value)

      File "/Users/sikaifeng/anaconda/lib/python3.6/site-packages/pandas/core/internals.py", line 2303, in _try_coerce_args
        raise TypeError

    TypeError
As you can see, every step before the crash, the program added and drop rows as I want. Even after the last drop, it went well, and I can see the DataFrame after dropping. And I can see the insert row I am about to add into the DataFrame, it is correct. But then when I try to access the DataFrame by trying to print it before actually inserting the row, it crashes. I think my program should be correct.

Is this a systematic bug? If not, how could I change my code. Thank you guys!
