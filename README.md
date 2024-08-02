# bolsa-real

import http.client as httplib
import urllib, base64, re
import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
import time
import numpy as np
from matplotlib.ticker import MultipleLocator, FormatStrFormatter
conn = httplib.HTTPSConnection('candle.etoro.com')
import pandas
import pandas as pd
def rsis(df, periods = 14, ema = True):
    """
    Returns a pd.Series with the relative strength index.
    """
    close_delta = df['close'].diff()

    # Make two series: one for lower closes and one for higher closes
    up = close_delta.clip(lower=0)
    down = -1 * close_delta.clip(upper=0)
    
    if ema == True:
	    # Use exponential moving average
        ma_up = up.ewm(com = periods - 1, adjust=True, min_periods = periods).mean()
        ma_down = down.ewm(com = periods - 1, adjust=True, min_periods = periods).mean()
    else:
        # Use simple moving average
        ma_up = up.rolling(window = periods, adjust=False).mean()
        ma_down = down.rolling(window = periods, adjust=False).mean()
        
    rsi = ma_up / ma_down
    rsi = 100 - (100/(1 + rsi))
    return rsi


def history(a):
    body = a[2:-4]
    print (body)
    close=0 
    ope=0
    high=0 
    low=0
    for key in body:
       if "Close" in key:
           c = key.split(':')
           close = float(re.findall('\d+\.\d+', c[1] )[0])
       if "Open" in key:
           c = key.split(':')
           ope = float(re.findall('\d+\.\d+', c[1] )[0])
       if "High" in key:
           c = key.split(':')
           high = float(re.findall('\d+\.\d+', c[1] )[0])
       if "Low" in key:
           c = key.split(':')
           low = float(re.findall('\d+\.\d+', c[1] )[0])
    return close, ope, high, low

def llamado():
    conn.request("GET", "/candles/asc.json/OneMinute/1/100000")
    #NVIDIA 1137
    #oil  17
    #palladium41
    # 100000 bitcoin

    ##https://github.com/Menox10/Etoro-history-downloader/blob/master/history_bitcoin.py
    response = conn.getresponse()
    data = response.read()
    a = data.decode().split(",")
    #closeprice = history(a)
    close, ope, high, low=history(a)
    
    return close, ope, high, low
    


plt.style.use('fivethirtyeight')
plt.rcParams["figure.figsize"] = (20,3)


o = []
l = []
z = []
h=[]
rsi=[]
relaciont=[]
linewidths=0.9
from os import system, name 
for x in range(0, 500):
    
    close, ope, high, low=llamado()
    z.append(close)
    o.append(ope)
    l.append(low)
    h.append(high)
    if len(z)>200:
      del o[0]
      del l[0]
      del h[0]
      del z[0]
      del relaciont[0]

    avg = np.mean(z)
    median=np.median(z)
    relacion=avg/median
    relaciont.append(relacion)
    medianrelaciont=np.mean(relaciont)
    plt.plot(z, label='Close',linewidth=linewidths)
    plt.plot(o, label='Open',linewidth=linewidths)
    plt.plot(h, label='High',linewidth=linewidths)
    plt.plot(l, label='Low',linewidth=linewidths)
    plt.axhline(y=median, xmin=0.1, xmax=0.9,label='Median', color='y',linewidth=linewidths)
    plt.axhline(y=avg, xmin=0.1, xmax=0.9,label='Avg',color='b',linewidth=linewidths)
    
    plt.legend(loc='upper left')
    plt.show()
  
    plt.close()

    plt.plot(rsi, label='RSI',linewidth=linewidths)
    if len(rsi)>16:
      print ("_______________=======",rsi)
      print ("_______________===ssss====",rsi[0])
      kiki=rsi[-1]
      print ("_______________=====OOOO==",kiki)
      print ("_______________=====OOOO==",kiki-5)
      if float(kiki)>10:


      #TO DO  a array of X=x and Y=RSI-1 apend and plot 



        plt.plot(x,rsi[-1],"^")
    
    
    plt.legend(loc='upper left')
    plt.show()
    plt.close()
    
    
    if medianrelaciont>1 :
      print("Comprar")
    else:
      print ("Vender")
    if len(z)>1 :
      print ("Apertura:",o[-1],"Cierre",z[-1],"H: ",h[-1],"L:",l[-1], "Anterior:",z[-2], "Relacion inmediata: ",relacion, "Relacion en el tiempo: ", medianrelaciont)
    #print ('esto es Z',z)
    b = [float(s) for s in z]
    #print (float(z)-3)
    if len(z)>=15:
      df = pd.DataFrame({'close': z})
      kiwi=rsis(df, periods = 14, ema = True)
      rsi.append(kiwi.iloc[[-1]])
    else:
      rsi.append(0)


    time.sleep(0.5)


    
conn.close()
