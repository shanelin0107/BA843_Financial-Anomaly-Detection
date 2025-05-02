# BA843_Financial-Anomaly-Detection  
## Introduction & motivation  
Money laundering threatens the integrity and stability of financial systems worldwide by disguising illicit proceeds as legitimate funds. In this analysis, we bring together transaction data from IBM, advanced anomaly detection techniques, and end-to-end implementation guidance to show you exactly how machine learning models can uncover hidden patterns of illicit behavior in real time. By reading this report, you will gain a clear understanding of the challenges faced by traditional review processes, learn which machine-learning models and features deliver the greatest detection accuracy, and see step-by-step code examples that make it straightforward to deploy and continuously improve your own AML pipeline.

Our motivation is threefold. First, as transaction volumes soar and laundering methods grow ever more sophisticated, manual review processes struggle to be efficiency and cost effectiveness. Second, intensifying global AML regulations impose heavy compliance burdens on financial institutions, which must balance risk management against operational costs to avoid penalties and reputational damage. Finally, we would like to apply waht we learn about machine-learning approaches to offer a scalable, self-learning defense: by automatically surfacing subtle or emerging laundering schemes, these technologies can transform AML from a reactive compliance exercise into a proactive safeguard.  

## Project Proposal
This project aims to utilize the IBM Anti-Money Laundering transaction dataset to analyze and identify anomalous behaviors within financial transactions, thereby aiding in the early detection of potential money laundering activities. We will:

1. Examine transaction patterns among different banks and accounts, assessing whether high-value or frequent transfers are concentrated in specific sources or destinations.

2. Compare transaction amounts, payment methods, and currency types used in laundering to identify whether specific payment formats (e.g., ACH, Credit Card, Cheques) or currencies (e.g., USD, Euro, Yuan) are more commonly associated with illicit transactions.

3. Analyze transaction amounts, frequency, and time-series variations to identify abnormal fluctuations or unusual transaction behaviors.

4. Apply supervised/unsupervised machine learning models to detect anomalous patterns in the data. To extract insight from the transactions.

5. Evaluate the effectiveness and accuracy of various machine learning models, such as logistic regression, random forest,K-means, and Isolation Forest in detecting anomalies in real-world anti-money laundering scenarios.

## Dataset Description

1. IBM AML Dataset
(Link: https://www.kaggle.com/datasets/ealtman2019/ibm-transactions-for-anti-money-laundering-aml)
- LI-Medium_Trans.csv (lower illicit ratio)
- HI-Medium_Trans.csv (higher illicit ratio)

| Column            | Description                                                                 |
|-------------------|-----------------------------------------------------------------------------|
| Timestamp         | Year/Month/Day and Hour:Minute of transaction time                          |
| From Bank         | Numeric code for bank where transaction originates                          |
| Account2          | Hexadecimal code for account where transaction originates                   |
| To Bank           | Numeric code for bank where transaction ends                                |
| Account4          | Hexadecimal code for account where transaction ends                         |
| Amount Received   | Monetary amount received in From account (in currency units of next column) |
| Receiving Currency| Currency used in From account (e.g., USD, EUR)                              |
| Amount Paid       | Monetary amount paid (in currency units of next column)                     |
| Payment Currency  | Currency used for payment (e.g., USD, EUR)                                  |
| Payment Format    | How transaction was conducted: cheque, ACH, wire, credit card, etc.         |
| Is Laundering     | 1 = laundering, 0 = not laundering                                          |

2. Currency.csv -- Self Created

| Column             | Description                                                              |
|--------------------|--------------------------------------------------------------------------|
| Payment Currency   | Name of the currency used for paying (e.g., US Dollar, Euro, Yen)        |
| Receiving Currency | Name of the currency used for receiving the transaction                  |
| ISO_pay            | ISO 4217 code for the payment currency (e.g., USD, EUR, JPY)             |
| Country_pay        | Country or region associated with the payment currency                   |
| Region_pay         | Geographic region of the payment country (e.g., Asia, Europe)            |
| ISO_recv           | ISO 4217 code for the receiving currency                                 |
| Country_recv       | Country or region associated with the receiving currency                 |
| Region_recv        | Geographic region of the receiving country (e.g., Asia, Europe)          |

## Executive Summary
In this project, our goal is to detect potential money laundering activities through transaction analysis. We begin by loading the dataset from our S3 bucket and performing an initial overview of its dimensions and missing values. In our case, we intentionally retain outliers, since they may represent the most suspicious transactions for our analysis. We then merge this main dataset with our currency exchange file.

Next, we carry out exploratory data analysis to uncover key patterns: the time-series behavior of transactions, transaction volumes by region, the currencies in use, and the payment formats most commonly employed. After identifying these insights, we move on to data preprocessing to prepare the data for our machine-learning models. This phase includes converting string fields to numeric values, encoding categorical variables, and standardizing datetime formats.

Once the data are ready, we train a baseline logistic regression model and follow by random forest classifier, using our labeled data for evaluation. We employ LIME to interpret and explain the models’ predictions. Finally, due to computational constraints, we sample the dataset and apply K-means clustering to identify distinct transaction groups, as well as Isolation Forest to compute an anomaly score for each transaction.  

## Conclusion
In this project, we first conducted an in-depth exploratory data analysis (EDA) to understand the fundamental patterns in the transaction data. We observed a sharp drop in daily transaction volumes after mid-September, suggesting possible data sparsity or logging issues. Weekly transaction patterns peaked on Thursdays and Fridays, aligning with typical business cycles, while an unusually high number of transactions occurred at midnight—raising suspicion of automated or scripted behaviors often associated with laundering activities. Although 98.5% of transactions were domestic, the 1.5% involving international flows carried outsized risk, especially concentrated among the United States, the European Union, and China. Currency-wise, most transactions stayed within the same denomination (USD→USD, EUR→EUR), and while cheques and credit cards dominated general payment activity, laundering-flagged transactions were almost exclusively routed through ACH transfers.

Building on these insights, we proceeded to supervised machine learning. We trained logistic regression and random forest models on the labeled dataset. Both models achieved high recall but relatively low precision—an intentional design choice reflecting the priorities of anti-money laundering work. In this field, missing a true laundering case (false negative) is far riskier than generating excess alerts (false positives). Therefore, our models were tuned to capture as many suspicious transactions as possible, even at the cost of flagging many normal transactions. To interpret model behavior, we applied LIME specifically to the logistic regression model. LIME results indicated that AUD-denominated payments (ISO_pay = AUD) were the most influential positive risk signal, followed by payment routes linked to the United Kingdom, Japan, and Saudi Arabia, while transaction time features showed relatively little contribution to risk prediction.

Due to computational limitations, we applied unsupervised learning on a sampled subset of the data. K-means clustering revealed several distinct transaction groups: Cluster 0 consisted of transactions with high cross-border activity, Cluster 1 was characterized by extremely large average payment amounts (around $7 million), while Clusters 2 and 3 represented more typical transactions. Isolation Forest further identified anomalies, predominantly flagging extremely large cross-border flows, such as North America to Asia transfers involving yen to USD conversions, as well as AUD→USD and CNY→USD transactions. These outliers, some exceeding tens of millions or even over a trillion in amount, could signal either genuine laundering activity or potential data entry errors.

## Recommendations:  

1. Channel-Based Monitoring: Prioritize ACH transactions, especially those denominated in Euros and AUD, for enhanced screening.

2. Temporal-Based Alerting: Heighten alert sensitivity during midday peaks and around midnight to capture suspicious timing patterns.

3. Tiered Alert System: Implement a two-tier risk categorization, where high-value cross-border flows and large AUD transactions form Tier 1 (highest priority), while UK-, Japan-, and Saudi-linked payments form Tier 2.

4. Cluster-Based Sampling: Randomly audit transactions in clusters showing unusually high payment amounts or international movement to validate model signals and detect possible blind spots.

5. Feature Enrichment: In future work, introduce dynamic behavioral features, such as transaction velocity, transaction-to-balance ratios, and account risk histories, to strengthen model performance and reduce false positives without compromising sensitivity.
 
