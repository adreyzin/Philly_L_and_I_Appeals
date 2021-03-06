# Philadelphia Licenses and Inspections Appeals Decision Results
<img src="images/appeal_map.png"/>  

## Problem Identification  
Create the best possible model to accurately predict the appeal decision. This prediction model can help Philadelphia homeowners filing appeals with the L&I board. 

## Data Collection, Organization, and Definitions  
This research will be based on the statistical data provided by OpenDataPhilly portal in the [L&I Appeals dataset](https://www.opendataphilly.org/dataset/license-and-inspections-appeals/resource/b721ad52-9e27-46d2-b494-6bf0ef1c7603).  
The Department of Licenses and Inspections accepts applications for appeals of various violations, refusals, revocations, and denials to the following Boards: Board of Building Standards, Licenses and Inspections (L&I) Review Board, and Zoning Board of Adjustments. The Board Decisions dataset shows the decisions made by the Appeal Boards (LIRB, ZBA, BBS).  
  
## Exploratory Data Analysis  
### Reducing number of decision outcomes
Currently the Decision column for the dataset has 21 various outcomes: 'admin/review', 'affirmed', 'approved', 'boardaknowl', 'continued', 'denied', 'dismissed', 'granted', 'held', 'held/info', 'issued', 'late-apprvd', 'late-denied', 'moot', 'newhearnot', 'newhearyes', 'refused', 'remand', 'reschedule', 'revised', 'sustained'. To simplify the modeling process these values were translated in only three outcomes: 'approved', 'denied', and 'other'.  
### Adding 'lawyer' feature  
It appears that the top appellants are all lawyers. This can’t be verified without access to the Philadelphia Bar Association records. 
An assumption has been made that ESQ or ESQUIRE in the field identifies the primary appellant as a lawyer.  
| appellant | No of Recs |
| ----- | ------------------------------------ |
|SHAWN D. WARD, ESQUIRE | 421|
|ZHEN JIN | 288|
|JOSEPH BELLER, ESQ. | 254|
|DAVID ORPHANIDES, ESQ. | 238|
|SHAWN WARD, ESQ. | 228|
|RONALD PATTERSON, ESQ. | 225|
|ALAN NOCHUMSON, ESQ. | 198|
|RUSTIN OHLER | 182|
|BEDITZA CADILLO | 170|
|BRETT D. FELDMAN, ESQUIRE | 165|
|VERN ANASTASIO, ESQ | 162|
|STEPHEN G. POLLOCK, ESQUIRE | 158|
|CARL PRIMAVERA, ESQUIRE | 150|
|LEO MULVIHILL, JR., ESQ. | 147|
|HENRY M. CLINTON, ESQ | 143|

### Addign landmarks for more feature generation
Following four landmarks were created to assist in creating additional features.
<img src="images/landmarks_map.png"/>  

### Appeals Map
The more densely populated areas of the city have higher density of appeals. City parks, airports and other non-populated areas do appear empty on the map as well.
Approved and Denied centroids are very close to each other.
<img src="images/appeal_map.png"/>  

### Distances from Center City
Histogram of distances from Center City shows that the most of the records are concentrated closer to Center City and the number gradually drops further away. This is also consistent with the Center City being more densely built and more densely populated.
<img src="images/distances_from_CC.png"  width="400">  
  
## Pre-processing and Training Data Development  
Following steps have been taken for the data pre-processing:  
* Application Type column had both fully spelled out values: 'Zoning Board of Adjustment', 'Board of Building Standards', and 'L&I Review Board Codes' as well as abbreviated values: 'RB_ZBA', 'RB_LIRB', 'RB_BBS'. Abbreviated values have been replaced with the fully spelled values and the column was replaced altogether with three "dummy" columns.  
* There were only 4 records for 'Plumbing Advisory Board'. They were dropped from the dataset.  
* Based on presence of a word 'esquire' or 'esq' in primaryappellant column new categorical column 'lawyer' has been created.  
* Following Philadelphia landmarks were added: Center City, South Philadelphia High School, Philadelphia County Assistance Office Delancey District and Frankford Transportation Center. New features have been created to measure the distance to these landmarks.  
* New features for the squared values of the distances to the landmarks were created as well.  
* Values in the column appealgrounds were all changed to the lower case and all punctuation marks have been removed.  
* Categorical applicationtype column has been replaced with the "dummy" quantifiable data.  
* Further TFIDF classifier has been used with appealgrounds column. Following parameters were used while building the classifier:  
  * English stop words were used  
  * n-gram range 1,3 (from unigram to tri-gram)  
  * Word document frequency was set to be between 15 and 75  

25% of the data were set as a test dataset with the rest used for model training.  

## Modeling  
Several models have been used with various feature selection with and without the results of the TFIDF vectorizer. It appears that the K Neighbors Classifier model without TFIDF vectorizer with the number of neighbors set to 10 has produced the most accurate results. The model was the most accurate while using the most but not all of the available features.  
Interestingly enough no model produced better results while using TFIDF vectorizer compared to the models without it. The best one with TFIDF vectorizer was Decision Tree Classifier model.  
In order to obtain a good balance between Precision and Recall measures F1 score was used as a primary test score.  

Here is the top 10 performing models that I have tried:  
| Model | Features | F1 score Test | F1 score Train | Accuracy Test | Accuracy Train | Precision Test | Precision Train | Recall Test | Recall Train |
| ----- | -------- | -------- | --------- | ---------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE', 'fromCC_Sq', 'fromSouth_Sq', 'fromWest_Sq', 'fromNE_Sq' | 0.708 | 0.735 | 0.742 | 0.765 | 0.707 | 0.744 | 0.742 | 0.765 |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE' | 0.707 | 0.735 | 0.741 | 0.765 | 0.706 | 0.744 | 0.741 | 0.765 |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE', 'from_approved_center', 'from_denied_center' | 0.707 | 0.735 | 0.742 | 0.764 | 0.706 | 0.743 | 0.742 | 0.764 |
| Decision Tree Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE', 'fromCC_Sq', 'fromSouth_Sq', 'fromWest_Sq', 'fromNE_Sq' **with TFIDF Vectorizer** | 0.707 | 0.709 | 0.756 | 0.757 | 0.722 | 0.734 | 0.756 | 0.757 |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE', 'fromCC_Sq', 'fromSouth_Sq', 'fromWest_Sq', 'fromNE_Sq', 'from_approved_center', 'from_denied_center' | 0.706 | 0.734 | 0.741 | 0.764 | 0.705 | 0.743 | 0.741 | 0.764 |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC' | 0.705 | 0.731 | 0.743 | 0.763 | 0.706 | 0.742 | 0.743 | 0.763 |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center' | 0.705 | 0.734 | 0.740 | 0.764 | 0.704 | 0.743 | 0.740 | 0.764 |
| K Neighbors Classifier | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE', 'fromCC_Sq', 'fromSouth_Sq', 'fromWest_Sq', 'fromNE_Sq', 'from_approved_center', 'from_denied_center', 'from_approved_center_Sq', 'from_denied_center_Sq' | 0.698 | 0.733 | 0.737 | 0.763 | 0.696 | 0.743 | 0.737 | 0.763 |
| Logistic Regression | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE' | 0.695 | 0.691 | 0.740 | 0.735 | 0.692 | 0.690 | 0.740 | 0.735 |
| Logistic Regression | 'Board of Building Standards', 'L&I Review Board Codes', 'Zoning Board of Adjustment', 'lawyer', 'fromCC', 'from_general_center', 'fromSouth', 'fromWest', 'fromNE', 'fromCC_Sq', 'fromSouth_Sq', 'fromWest_Sq', 'fromNE_Sq' | 0.693 | 0.687 | 0.736 | 0.730 | 0.696 | 0.693 | 0.736 | 0.730 | 


