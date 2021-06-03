---
title: 使用prophet预测销售额
author: 宇飞的世界
date: '2021-04-14'
slug: prophet
categories:
  - R package
tags:
  - prophet
  - sales predict
---





## 前言

留意到"先知"prophet已经有一段时间，第一次看到先知介绍时候注意到可以人为指定异常点以及节假日让我感觉到很新颖，以往在使用**forecast**包预测销售额时没有该功能。

先知可以在R中和Python中实现预测，并且速度快，提供自动化的预测，或手工调整。官网上的介绍：

Prophet is a procedure for forecasting time series data based on an additive model where non-linear trends are fit with yearly, weekly, and daily seasonality, plus holiday effects. It works best with time series that have strong seasonal effects and several seasons of historical data. Prophet is robust to missing data and shifts in the trend, and typically handles outliers well.

Prophet is [open source software](https://code.facebook.com/projects/) released by Facebook’s [Core Data Science team](https://research.fb.com/category/data-science/). It is available for download on [CRAN](https://cran.r-project.org/package=prophet) and [PyPI](https://pypi.python.org/pypi/prophet/).

[prophet官网](https://facebook.github.io/prophet/)，本文代码在R中实现，主要是翻译官网中的案例。



## Prophet介绍



### 安装

prophet在[CRAN](https://cran.r-project.org/package=prophet)上，可以直接下载。

```R
# R
> install.packages('prophet')
```

在Windows上，R需要编译器，因此您需要[按照所](https://github.com/stan-dev/rstan/wiki/Installing-RStan-on-Windows)提供[的说明](https://github.com/stan-dev/rstan/wiki/Installing-RStan-on-Windows)进行操作`rstan`，关键步骤是在尝试安装软件包之前先安装[Rtools](http://cran.r-project.org/bin/windows/Rtools/)。具体参照官网上的安装介绍。

### 快速使用

首先获取数据，先知中数据结构data.frame中需要ds、y列，其中ds列是日期，y列值。代码中的数据为包提供的演示案例，可在此处[下载](https://github.com/facebook/prophet/blob/master/examples/example_wp_log_peyton_manning.csv)

```
# R
library(prophet)
library(data.table)
df <- fread('https://raw.githubusercontent.com/facebook/prophet/master/examples/example_wp_log_peyton_manning.csv')
```

我们调用 `prophet` 函数去拟合模型，第一参数是历史数据，其他参数来控制拟合情况，参数在本文后面介绍。

```R
# R
m <- prophet(df)
```

使用`make_future_dataframe`函数指定预测时间周期进行并生成数据。默认情况下，生成的数据包含历史数据。

```R
# R
future <- make_future_dataframe(m,periods = 365,freq = 'day',include_history = TRUE)
tail(future)
```

使用`predict`函数来获取预测数据，生成的数据包含预测列，具有不确定性列，季节性成分列。

```
# R
forecast <- predict(m, future)
tail(forecast[c('ds', 'yhat', 'yhat_lower', 'yhat_upper')])
```

最后通过`plot`来绘制预测图

```
# R
plot(m, forecast)
```

使用该`prophet_plot_components`功能查看预测，分为趋势，每周季节性和年度季节性。

```
# R
prophet_plot_components(m, forecast)
```

可以使用dyplot.prophet(m, forecast)画动态图。

### Prophet介绍

本部分主要介绍prophet函数的参数，节假日效应，不确定性等

#### prophet函数参数

```R
prophet(
  df = NULL,
  growth = "linear",
  changepoints = NULL,
  n.changepoints = 25,
  changepoint.range = 0.8,
  yearly.seasonality = "auto",
  weekly.seasonality = "auto",
  daily.seasonality = "auto",
  holidays = NULL,
  seasonality.mode = "additive",
  seasonality.prior.scale = 10,
  holidays.prior.scale = 10,
  changepoint.prior.scale = 0.05,
  mcmc.samples = 0,
  interval.width = 0.8,
  uncertainty.samples = 1000,
  fit = TRUE,
  ...
)
```

- changepoint_prior_scale : 最重要的参数，它确定趋势的灵活性，尤其是确定趋势在趋势更改点处的更改量。默认值为0.05但是可以对其进行调整。[0.001，0.5]的范围大概是正确的。
- seasonality_prior_scale : 此参数控制季节性的灵活性，较大的值使季节性适应较大的波动，较小的值减小季节性的幅度，默认值10，调整的合理范围可能是[0.01,10],当设置为0.01时，季节性幅度非常小
- holidays_prior_scale : 该参数可以控制适应节日效果的灵活性，与seasonality_prior_scale类似，默认为10.0，调整的合理范围可能是[0.01,10]范围内。
- seasonality_mode : 选项为[ `'additive'`，`'multiplicative'`]，默认值为`'additive'`。根据业务的时间序列选择加法或乘法的季节性。
- changepoint_range : 允许更改趋势的历史记录的比例，默认值为历史记录的80%，最好不用调整此参数，[0.8，0.95]可能是一个合理的范围。
- growth : 选项为[ ‘linear’ and ‘logistic’]
- changepoints : 用于手动指定更改点的位置，默认情况下为None,模型自动放置突变点。
- n_changepoints :自动放置的更改点突变点的数量，默认值25
- yearly_seasonality : 默认情况下"auto",如果有一年的数据，这将打开每年的季节性，参数选项为['auto'，True，False]。
- weekly_seasonality : 与 yearly_seasonality相同。
- daily_seasonality : 与yearly_seasonality相同。
- holidays : 该参数接受指定的节假日数据框。
- mcmc_samples : 是否使用MCMC可能由时间序列的长度和参数不确定性的重要性等因素决定
- interval_width : prophet 返回预测的不确定性区间，例如`yhat_lower`和`yhat_upper`预测`yhat`。默认值0.8，如果希望间隔为95％，则可以将其更改为0.95。仅影响不确定性间隔，不会更改预测`yhat`，因此不需要调整。



关于参数的解释详情请参见官网。



#### 节假日影响

##### 指定节假日

通过指定节假日的数据框，如下指定元旦，同样方式可以指定春节、妇女节、清明节、劳动节、端午节、中秋节、国庆节、双11等节日，或自己公司设定的促销活动节日等。

```R
# R
> library(tidyverse)
> yuandan <- tibble(holiday = 'yuandan',
                  ds = ymd(c('2017-01-01','2018-01-01','2019-01-01','2020-01-01','2021-01-01')),
                  lower_window = 0,upper_window = 3)

```

除了上述的指定holiday、ds、lower_window、upper_window之外还能新增prior_scale列，为每个节假日单独指定"prior scale"。

创建节假日数据框后,通过holidays参数传递；如果发现假日过拟合,可以调整此参数，`holidays.prior.scale`默认参数为10

```R
# R
m <- prophet(df, holidays = holidays)
# m <- prophet(df, holidays = holidays, holidays.prior.scale = 0.05)
forecast <- predict(m, future)
```

节假日效果可以在`forecast`数据中查看

```R
# R
forecast %>% 
  select(ds, playoff, superbowl) %>% 
  filter(abs(playoff + superbowl) > 0) %>%
  tail(10)
```

可以通过`plot_forecast_component`函数查看某节假日的效应

```R
plot_forecast_component(m, forecast, 'chunjie')
```

##### 内置假期

可以使用`add_country_holidays`函数使用内置的节假日，但是不符合我们国家目前的现状，拿五一劳动节来说，有些年份是3天有些年份是5天，除夕前后销售也会有较大的影响。

```R
# R
m <- prophet(holidays = holidays)
m <- add_country_holidays(m, country_name = 'US')
m <- fit.prophet(m, df)
```

可以通过`train.holiday.names`查看包含哪些假期

```R
# R
m$train.holiday.names
```



#### 不确定区间

##### 趋势不确定

预测中最大的不确定性来源未来趋势变化的可能性，不可能很确定的知道。

不确定区间的宽度（默认为80％）可以使用以下参数设置`interval_width`：

```R
# R
m <- prophet(df, interval.width = 0.95)
forecast <- predict(m, future)
```

##### 季节不确定性

本段个人不太理解。默认情况下，模型仅返回趋势和观察噪声中的不确定性。为了获得季节性的不确定行，我们必须进行完整的贝叶斯采样。使用参数`mcmc.samples`(默认为0)可以完成此操作。

```R
# R
m <- prophet(df, mcmc.samples = 300)
forecast <- predict(m, future)
```

详情请参考官网

#### 诊断预测和真实差异

使用`cross_validation`功能，针对一系列历史日期自动完成交叉验证。通过指定预测的范围(horizon)，指定初始训练周期的大小(initial)和截止日期之间的间隔(period)，一半情况下initial是horizon的三倍，并且每半个周期进行一次截断。

```R
# R
df.cv <- cross_validation(m, initial = 730, period = 180, horizon = 365, units = 'days')

```

我们进行交叉验证，以评估在365天的范围内的预测效果，从第一个截止日期开始的730天训练数据开始，然后每180天进行一次预测。



## Prophet实战

通过上述介绍，我们大概知道prophet是怎样预测的。接下来，我们就公司实际业务数据预测，代码是R版本。因为目前所在公司是传统零售行业，商品销售情况有较强的季节性。

### 异常数据处理

先知prophet能处理历史数据中异常值，但只能通过使它们适应趋势变化来解决。处理离群值的最佳方法是将其删除，先知对丢失数据没有任何影响，将异常值设置为NA即可。

考虑到2020年的新冠疫情，将历史销售数据中的2020年1月24 日至4月30日的数据设置为NA

```R
dt[, y := fcase(
  ds < as_date("2020-01-24"), y,
  ds > as_date("2020-04-30"), y,
  default = NA
)]
```

### 预测

```R
m <- prophet(df = dt,holidays = holidays,seasonality.prior.scale = 8,holidays.prior.scale = 8,
             changepoint.prior.scale = 0.5,interval.width = 0.95) 
future <- make_future_dataframe(m,periods = 365)

forecast <-  predict(m,future)
```

#### 预测拟合效果

```R
# R
> plot(m, forecast)
```

#### 各成分走势图

```R
# R
> prophet_plot_components(m, forecast)
```

#### 查看节假日

```R
# R
> plot_forecast_component(m, forecast, 'chunjie')
```




