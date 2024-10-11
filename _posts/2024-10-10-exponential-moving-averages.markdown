---
layout: post
title:  "Exponentially Weighted Moving Averages and MACD"
date:   2024-10-10 00:00:00 -0700
tags: python
---

This code assumes that you have a Pandas DataFrame with a `price` column, and
a `datetime` column. The following code creates two columns, `ewm_12` and
`ewm_26` using the Pandas' `ewm` method, and the `span` argument. This will
create exponential moving averages over 12 and 26 day windows, respectively.

From there, we can calculate the Moving Average Convergence/Divergence curve by
taking the difference of the `ewm_12` and `ewm_26` curves. From there, the MACD
Signal is the 9 day window exponential moving average of the MACD curve.

{% highlight python %}
import matplotlib.pyplot as plt
import pandas
import seaborn

# build mkt dataframe with 'price' and 'datetime' columns

mkt['ewm_12'] = mkt['price'].ewm(span=12).mean()
mkt['ewm_26'] = mkt['price'].ewm(span=26).mean()
mkt['macd'] = mkt['ewm_12'] - mkt['ewm_26']
mkt['macd_signal'] = mkt['macd'].ewm(span=9).mean()
mkt['macd_minus_signal'] = mkt['macd'] - mkt['macd_signal']
{% endhighlight %}

From here, we can plot the `price` curve, and the exponential moving average
curves.

{% highlight python %}
m = mkt[['price', 'ewm_12', 'ewm_26']]
axes = seaborn.lineplot(data=m, color='k')
axes.grid(True)
plt.xticks(rotation=45)
plt.show()
plt.savefig('exponential-moving-average.png', dpi=200)
{% endhighlight %}

![Exponential Moving Averages](/assets/images/exponential-moving-average.png)

Here, we can plot the MACD curve, and the difference between the MACD curve and
the MACD Signal curve. On interpretation of this difference is that nagative
values suggest a bear market, while positive values suggest a bear market. 

{% highlight python %}
m = mkt[['macd', 'macd_signal', 'macd_minus_signal']]
axes = seaborn.lineplot(data=m, color='k')
axes.grid(True)
plt.axhline(y=0, color='k', linestyle='-')
plt.xticks(rotation=45)
plt.show()
plt.savefig('macd-signal.png', dpi=200)
{% endhighlight %}

![MACD and MACD Signal](/assets/images/macd-signal.png)