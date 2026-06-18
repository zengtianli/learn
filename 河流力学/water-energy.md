---
public: true
title: "浙东引水工程受水区降雨趋势与多尺度变率分析"
tags: [水利工程]
---

### Rainfall Trends and Multi-scale Variability in the Water Receiving Area of the Eastern Zhejiang Water Diversion Project

**ZENG Tian-li$^1$, ZUO Xiao-xia$^1$, YANG Yu$^1$, DH$^1$, WU Mu-hong$^2$, ZHONG Lü-bin$^2$, CHEN Shu-yang$^3$**
*(1. Zhejiang Design Institute of Water Conservancy and Hydroelectric Power Co., LTD., Hangzhou 310002, China; 2. Qingyuan County Water Conservancy Bureau, Lishui, Zhejiang 323800, China; 3. Zhejiang Yansi Information Technology Co., Ltd., Hangzhou, Zhejiang 310051, China)*

**Abstract:** To reveal the rainfall change patterns in the water receiving area of the Eastern Zhejiang Water Diversion Project and optimize project operation, this study analyzed long-term daily rainfall data from 1961 to 2022 for 15 typical sub-regions under the project's influence. Using methods such as the Mann-Kendall trend test, Sen's slope estimation, and multi-scale sliding window analysis, the spatiotemporal distribution, long-term trends, multi-scale variability, and regional correlation of rainfall were systematically investigated. The study found that annual rainfall in all sub-regions showed a significant upward trend (), and Hurst exponent analysis () indicated persistence in this upward trend; however, the magnitude of increase varied spatially (coastal > river basin > hilly areas), and interannual fluctuations intensified after 2010. Rainfall characteristics exhibit significant scale dependence: short time scales (3 months) show high variability and strong spatial heterogeneity, while long scales (12 months) are more stable with higher regional coherence. Inter-regional rainfall correlation generally strengthens with increasing analysis time scale, but uncertainty exists in the trend magnitudes estimated by Sen's slope. This study meticulously characterizes the long-term and multi-scale change features of rainfall in the study area, providing a crucial scientific basis for the adaptive scheduling of the Eastern Zhejiang Water Diversion Project and optimal management of regional water resources.

**Keywords:** Eastern Zhejiang Water Diversion Project; rainfall; spatiotemporal variation; trend analysis; multi-scale analysis; scale dependence

---

### 1. Introduction

The Eastern Zhejiang Water Diversion Project is the largest cross-basin water transfer project in Zhejiang Province, with an average annual diversion volume of 890 million m³. Since its operation began in 2014, it has significantly alleviated the contradiction between regional water supply and demand, providing essential water resource security for economic and social development along the route. The project involves multiple regions including Hangzhou, Shaoxing, and Ningbo, spanning different hydro-climatic units. The rainfall characteristics of its water sources, conveyance lines, and water receiving areas are complex and diverse. In recent years, rainfall patterns in the Eastern Zhejiang region have been undergoing significant changes. While annual precipitation shows an overall increasing trend, regional differences are obvious, and extreme precipitation events occur frequently. This directly affects the water source replenishment, conveyance safety, and supply-demand balance of the receiving areas, posing severe challenges to the scientific scheduling of the project and the optimal allocation of regional water resources.

Currently, although some studies focus on rainfall changes in Zhejiang Province and neighboring regions , most prioritize provincial scales, single basins, or specific seasons, with varying emphasis on data timeliness and spatial resolution. For the Eastern Zhejiang Water Diversion Project, a specific large-scale project, there is a lack of in-depth analysis on the spatiotemporal patterns, long-term trends, and multi-scale variability of rainfall within its extensive influence area. This makes it difficult to fully meet the urgent needs for refined and adaptive project scheduling.

In view of this, this study selects 15 typical sub-regions under the influence of the Eastern Zhejiang Water Diversion Project as research objects. Using long-term daily rainfall observation data from 1961 to 2022 (62 years), and employing Mann-Kendall trend tests, Sen’s slope estimation, Hurst exponents, and sliding window analysis methods, the study systematically parses the spatiotemporal distribution laws, evolutionary trends, and change characteristics at different time scales (monthly, semi-annual, annual) and regional correlations of rainfall in the project's receiving areas. The aim is to reveal the details of rainfall changes in the project's receiving areas under the background of climate change, providing a scientific basis for optimizing the scheduling strategies of the Eastern Zhejiang Water Diversion Project and improving regional water resource management efficiency and flood control and disaster reduction capabilities.

### 2. Study Area and Data

#### 2.1 Overview of the Study Area

The study area is located in the eastern part of Zhejiang Province (Fig. 1), covering 15 typical sub-regions within the five main zones of Xiaoshan, Shaoyu, Yuyao, Cixi, and Ningbo, which are primarily influenced by the Eastern Zhejiang Water Diversion Project. The multi-year average precipitation in the region ranges from 1273.5 mm to 1454.8 mm, with large inter-annual variations; the maximum annual precipitation can reach 2050.1 mm, while the minimum is only 682.2 mm. The region contains numerous water conservancy projects, mostly regulated by sluices and dams, forming a differentiated pattern of water resource utilization and flood control. It is a typical area for studying rainfall changes and their impacts.

**Fig. 1 Study area**

#### 2.2 Data Sources and Processing

The rainfall data used in this study are daily rainfall observation records from 1961 to 2022 for 15 representative stations within the study area. To ensure data quality, strict preprocessing was conducted: time series gaps were identified and processed through continuity tests; the  criterion (Pauta criterion) was used to identify and eliminate outliers; a small amount of missing data was interpolated using data from stations with the best correlation; and homogeneity tests and corrections were performed on data series using methods such as the sliding t-test to ensure reliability and consistency.

### 3. Methodology

#### 3.1 Mann-Kendall (MK) Trend Test

The non-parametric Mann-Kendall (MK) test method was used to determine the significance of monotonic trends in rainfall time series. This method does not require data to follow a specific distribution and is insensitive to outliers. It has been widely applied in regional rainfall change analysis in China and verified in related studies.
*(Equations 1-4 omitted for brevity, standard MK test formulas apply as per source)*
When , it indicates a significant trend at the 0.05 significance level.

#### 3.2 Sen's Slope Estimation

Sen's slope method was adopted to estimate the magnitude of the trend. This method determines the trend value by calculating the median of the slopes of all point pairs in the series. Combining Sen's slope estimation with the MK test is a commonly used robust method for assessing trends in hydro-meteorological series.
*(Equations 5-6 omitted)*
 indicates an upward trend, and  indicates a downward trend.

#### 3.3 Sliding Window Analysis

To explore the change characteristics and regional correlations of rainfall at different time scales, the sliding window analysis method was used. Three window widths () of 3 months, 6 months, and 12 months were selected to slide across the daily rainfall time series. Statistical characteristics were calculated for the data within each window, including:

* **Coefficient of Variation:** 
* **Linear Trend Slope within Window:** (Calculation based on least squares method)

By analyzing the changes in these statistics over time, the short-term, medium-term, and long-term fluctuation characteristics of rainfall are revealed.

#### 3.4 Hurst Exponent Analysis

The Hurst exponent () is an indicator measuring the long-term memory or persistence of a time series, often used to judge whether the future trend of a series will continue the past trend (persistence) or reverse (anti-persistence).
*(Calculation steps using Rescaled Range (R/S) analysis omitted)*
The value range of the Hurst exponent is . When , it indicates that the time series has long-term persistence; when , it indicates anti-persistence; when , the series is random.

### 4. Results and Analysis

#### 4.1 Rainfall Spatial Distribution Characteristics

Based on the analysis of daily rainfall data from 1961 to 2022, the multi-year average daily rainfall in the study area presents a significant spatial differentiation pattern. Overall, it features a "high in the center, low in the west, and medium in the east" pattern, with daily average rainfall ranging from 3.47 mm to 5.68 mm.

#### 4.2 Evolution of Long-term Rainfall Trends

**4.2.1 Trend Significance and Spatial Differentiation**
The Mann-Kendall (MK) test results (Table 1) show that during the 1961-2022 study period, the annual rainfall in all 15 sub-regions showed a statistically extremely significant upward trend. The magnitude of the MK test  value reflects the relative strength of the trend, presenting obvious spatial differences: the upward trend is strongest in the coastal Nansha Plain Area (), while it is relatively weaker in the inland Shaoyu Plain Area ().

**Table 1 Mann-Kendall Test Analysis Results** 
*(See Table 1 in source document for full data)*
Key Findings: Nansha Plain (, Sig.), Cixi Plain Midstream (, Sig.), Shaoyu Plain (, Sig.). All regions show significant upward trends ().

**4.2.2 Decadal Changes and Recent Fluctuations**
From the perspective of inter-annual changes (Fig. 2), the Eastern Zhejiang region as a whole has experienced significant decadal fluctuations. For example, 1967, 1978-1979, and 2003 were widespread rainfall troughs; while 1973, 1989, 2012, 2015, and 2019-2021 were high rainfall periods for multiple regions. Since 2010, both the frequency and amplitude of inter-annual rainfall fluctuations have shown signs of increasing, with rainfall increases being particularly prominent in areas such as the Yuyao Plain Mazhu Midstream Area, Yubei Plain Midstream Area, and Fenghui Plain Area.

**Fig. 2 Annual trend change chart**

**4.2.3 Spatiotemporal Characteristics of Trend Intensity (Sen's Slope)**
Sen's slope estimation further quantified the rate of change in annual rainfall for each sub-region (Fig. 3). Results show that the spatial pattern of trend intensity features "high in coastal areas, medium in river basins, and low in hilly mountains". The coastal Cixi Plain Midstream Area ( mm/year) and Nansha Plain Area ( mm/year) have the largest increases; central river receiving areas such as the Yuyao Plain Yaojiang Downstream Area ( mm/year) have moderate growth rates; while the Shaoyu Plain Area ( mm/year) and Yubei Plain Upstream Area ( mm/year), influenced by hilly terrain, have relatively the lowest growth rates. Fig. 4 shows that although the Sen's slope value for some areas (like Yuyao Plain Mazhu Midstream Area) is not the highest ( mm/year), its baseline rainfall is significantly higher than other regions, and the inter-annual fluctuation range is huge (approx. 1750-3250 mm).

**Fig. 3 Spatial pattern of rainfall in eastern Zhejiang**
**Fig. 4 Trend analysis of the Mazhu Middle River Area, Yuyao Plain**

**4.2.4 Trend Uncertainty Analysis**
Analysis of the 95% confidence intervals for Sen's slope estimates (Fig. 5) reveals that the confidence intervals for all sub-regions include zero. This indicates that although the MK test detected statistically significant long-term upward trends, considering the high inherent inter-annual variability of rainfall, the possibility of the trend being zero cannot be completely ruled out at the 95% confidence level. The width of the confidence intervals also reflects differences in rainfall variability across regions; for instance, the Yuyao Plain Mazhu Midstream Area has the widest confidence interval, consistent with its largest inter-annual fluctuations.

**Fig. 5 Rainfall change trend and 95% confidence interval in eastern Zhejiang**

**4.2.5 Persistence Analysis of Rainfall Trends**
To explore the future persistence of the identified upward trends in annual rainfall, the R/S analysis method was used to calculate the Hurst exponent () for the annual rainfall time series (1961-2022) of each sub-region. The calculation results are detailed in Table 2.

**Table 2 Calculated Hurst exponents of annual rainfall for each sub-region** 

* **Nansha Plain Area:** 0.6257
* **Yuyao Plain Yaojiang Downstream Area:** 0.6091
* **Cixi Plain Midstream Area:** 0.5890
* *(All regions have )*

As seen in Table 2, the Hurst exponents for annual rainfall in all 15 sub-regions within the study area are greater than 0.5, ranging from 0.5595 (Yuyao Plain Mazhu Midstream Area) to 0.6257 (Nansha Plain Area). The annual rainfall changes in each sub-region all show varying degrees of positive persistence. This implies that the observed upward trends in rainfall are likely to continue for some time in the future.

#### 4.3 Multi-time Scale Variation Characteristics of Rainfall

Using the sliding window analysis method, the change characteristics of rainfall on three time scales (3 months, 6 months, and 12 months) were examined, selecting the Cixi Plain East River Area as a representative region (Fig. 6).

* **3-Month Scale:** Rainfall in the Cixi Plain East River Area shows extremely high instability. The coefficient of variation fluctuates violently, frequently exceeding 1.0 with peaks above 1.5; the corresponding trend slope also shows large-amplitude positive and negative oscillations, ranging roughly between -5 and +5 mm. This indicates that intra-seasonal rainfall and short-term trends are very significant and difficult to predict.


* **6-Month Scale:** The stability of rainfall increases. The fluctuation amplitude of the coefficient of variation is reduced compared to the 3-month scale, mostly between 0.5 and 1.5. Meanwhile, the fluctuation range of the trend slope narrows significantly, roughly between -2 and +2 mm, showing a steadier changing trend.


* **Annual Scale:** Rainfall changes are the most gentle and stable. The coefficient of variation mostly fluctuates between 0.4 and 1.0, with an overall level lower and fluctuations smaller than the previous two scales. The change in trend slope is very weak, with a fluctuation range of only about -0.5 to +0.5 mm. The series clearly presents decadal variation characteristics, such as relative highs or lows in the early 1990s, early 2000s, and mid-2010s. This suggests that rainfall changes on the annual scale are mainly regulated by longer-period climatic factors.



**Fig. 6 Sliding window analysis (3-month, 6-month, and 12-month scales) for the Donghe Area, Cixi Plain**

#### 4.4 Scale Dependence of Regional Rainfall Correlation

By calculating the correlation coefficients of rainfall series in different sub-regions under the same sliding windows (3 months, 6 months, 12 months) (Fig. 7), the scale-dependent characteristics of inter-regional rainfall synergy were analyzed.

Results indicate that as the analysis time scale increases, inter-regional rainfall correlations generally strengthen. The average correlation coefficient for all regional pairs increased from 0.83 at the 3-month scale to 0.87 at the 12-month scale. This indicates that on long time scales, rainfall patterns in the Eastern Zhejiang region are increasingly controlled by regional factors and show better synergy; whereas on short time scales, they reflect more local randomness and spatial heterogeneity. Regions of different geographical types show differentiated correlation evolution patterns:

* **Coastal Regions:** Internal correlation significantly strengthens with increasing time scale (average rising from 0.82 to 0.86). For example, the correlation between the Midstream Area and West River Area of the Cixi Plain increased from 0.90 to 0.94, reflecting stronger consistency in marine climate influence on the annual scale.


* **River and Inland Plain Regions:** Internal correlation change patterns are relatively complex. For instance, the correlation between the Upstream and Downstream areas of the Yaojiang Basin continued to strengthen with scale (0.92  0.94), reflecting strengthened hydrological connections. However, the correlation between the geographically unique Yuyao Plain Mazhu Midstream Area and most other regions presents a unique "increase then decrease" pattern (e.g., with Nansha Plain Area: 0.71  0.72  0.68), which may be related to its specific hydro-geographic conditions.


* 
**Hilly and Mountainous Regions:** The increase in internal correlation is relatively small (average rising from 0.83 to 0.85), indicating that rainfall patterns in these regions are subject to strong and relatively stable terrain effects across different time scales.



**Fig. 7 Inter-regional correlation under 3-month, 6-month, and 12-month sliding windows**

### 5. Conclusion

This study systematically revealed the spatiotemporal evolution characteristics of rainfall in 15 influence sub-regions of the Eastern Zhejiang Water Diversion Project from 1961 to 2022. Main conclusions:

1. Annual rainfall in all regions shows a **significant upward trend** (MK test ). Hurst exponent () analysis indicates that this trend is persistent, but the magnitude of increase has significant spatial differentiation (Sen's slope: Coastal > River Basin > Hilly), and inter-annual fluctuations have intensified since 2010.


2. Rainfall characteristics have obvious **scale dependence**: short time scales (3 months) are unstable with strong spatial heterogeneity, while long scales (12 months) are more stable with higher spatial synergy. Inter-regional rainfall correlation patterns vary complexity with scale.


3. These refined rainfall change characteristics provide a key scientific basis for optimizing the multi-spatiotemporal scheduling of the Eastern Zhejiang Water Diversion Project and addressing regional water supply and flood control risks in a changing environment. Future research combining extreme precipitation indices and hydrological models is recommended.
