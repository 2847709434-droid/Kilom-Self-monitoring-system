# Research Log

---

## 2026-02-26 — Physio Data Exploration (Feb 16–19)

### Objective

Explore physiological signals collected during family gatherings and outings.

---

### Code Learning

```python
#----------------------Data preparation-------------------------------
import matplotlib.dates as mdates  # format x-axis to hour level
import seaborn as sns              # statistical visualization built on matplotlib

path = r"..."
#Raw string：trear backslashes as literal characters

tree = ET.parse(path)
root = tree.getroot()
#Extract XML root element

df = pd.DataFrame(
    [list(attrib.values()) for attrib in root.findall('Record')]
)
#(Background:Most HR,step, and sleep records in Apple Health data are stored as <Record.../>elements in XML format)

df['startDate'] = pd.to_datetime(df['startDate']).dt.tz_localize(None)
df['endDate'] = pd.to_datetime(df['endDate']).dt.tz_localize(None)
#pd.to_datetime ->Convert string tinmestamps into datetime objects
#.dz.tz.localize(None)->Remove timezone information for consistent slicing

#------------Target Time Filtering------------------

start_time=pd.to_datetime('2026-02-16 17:00:00')
end_time = pd.to_datetime('2026-02-19 23:00:00')
#Convert boundary timestamps into datetime objects

mask = (df['startDate'] >= start_time) & (df['endDate'] <= end_time)
#boolean mask for selecting records within the target time range

#-------------Target Type Filtering------------------

RecordDF=df.loc[mask].copy()
RecordDF['value']=pd.to_numeric(RecordDF['value'],errors='coerce')
#Convert value column to numeric type
#errors='coerce' -> converts non-numeric entries into NaN

target_signals={
    'HRV': 'HKQuantityTypeIdentifierHeartRateVariabilitySDNN',
    'HR': 'HKQuantityTypeIdentifierHeartRate',
    'Respiratory': 'HKQuantityTypeIdentifierRespiratoryRate',
    'Oxygen': 'HKQuantityTypeIdentifierOxygenSaturation',
    'Audio': 'HKQuantityTypeIdentifierEnvironmentalAudioExposure'
}
# Dictionary mapping readable signal names to Apple Health identifiers

#-------------------Visualization---------------------

fig, axes = plt.subplots(5, 1, figsize=(12, 20), sharex=True)
#Create stacked subplots with shared x-axis

for i, (name, identifier) in enumerate(target_signals.items()):
#.items ->seperate the key and value ，enumerate:auto-numbering
#循环轮次, i (序号), name (键),    identifier (值/长ID),   对应画布房间
#第 1 圈,  0,       'HRV',        'HK...SDNN',            axes[0] (最顶上的图)
#第 2 圈,  1,       'HR',         'HK...HeartRate',       axes[1] (第二张图)
#第 3 圈,  2,       'Respiratory','HK...RespiratoryRate', axes[2]
#...,...,...,...,...
    data = RecordDF[RecordDF['type'] == identifier]
    sns.scatterplot(data=data, x='startDate', y='value', ax=axes[i], s=15, alpha=0.6)
#Scatterplot used to inspect missing patterns and distribution

    axes[i].xaxis.set_major_locator(mdates.HourLocator(interval=2))
#Show x-axis ticks every 2 hours
    axes[i].xaxis.set_major_formatter(mdates.DateFormatter('%m-%d %H:%M'))
    
    axes[i].set_title(f'Raw Data: {name}', fontsize=14)
    axes[i].set_ylabel('Value')
    

    axes[i].tick_params(axis='x', rotation=45) 
#rotate x-axis labels to prevent overlap

plt.tight_layout()
#Automatically adjust subplot spacing
plt.show()

#Matplotlib automatically leaves slight padding outside the selected time window

```
### Problems in Apple Watch auto-sampling
Apple Watch Official Sampling Standards：

- HRV (SDNN) documentation: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/heartratevariabilitysdnn?utm_source=chatgpt.com
- Heart Rate:
https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/heartrate?utm_source=chatgpt.com
- Respiratory Rate:
https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/respiratoryrate?utm_source=chatgpt.com


#### Inaccurate sampling dute to ADHD-related movement tendency
##### Probelms 1:lack of reliable baseline recording
###### Solution 1:establish a standardlized morning baseline protocol:
within 5 minitutes after waking
remain lying in bed
record 3-5 minutes of:HR,HRV,Respiratory Rate,Audio
Maintaing natural breathing(no intentional regulation)
###### Double-Baseline stucture(if sleeping variability is significant)

##### Problems 2:
HRV(SDNN based):recorded only during steady states, often once evert few hours

<img width="1020" height="333" alt="ce25d9aad47f9c0f156f9fb5afd5cf86" src="https://github.com/user-attachments/assets/bd181fb2-3805-422e-9ec4-5855479160f0" />

HR:irregular and context-dependent sampling frequency

<img width="1029" height="333" alt="976b923456ecefabae0f2b83f61c6e65" src="https://github.com/user-attachments/assets/b5820378-1d69-486f-8600-c2f5f41b36b0" />

Respiratory Rate:Recorded sporadically under resting conditions

<img width="1020" height="333" alt="49bce72f716093728eb905d63873a22e" src="https://github.com/user-attachments/assets/2f481159-812e-4f64-8765-15d9afafb88d" />

###### Solutions 2:Manual Forced Recording via Zhengnian App
When entering a relevent state:
    Manually open Zhengnian App
    Remain physically steady for 1-5 minutes
    Ignore initial transitional data
    Maintain natural physiological rhythm
Rationale:Physiology changes are continuous rather than abrupt,A 1-5 minute steady recording window introduces accepteble error.
Forced recording offer "Valuable states"labels.

### Additional Vairable:Wearing earpods

## 2026-02-27 — Physio Data Exploration (Feb 16–19)-Sparse Data Analysis
- Despite the lack and inconsistency of the data, the dataset has its adventage in detailed and full sujective records and pysiological state description. It will be more effective to apply the methond of Event-based analysis than Time Series Modeling.
### Why windows?
- Fixed window and sliding window
- Noise reduction: Physiological signals contain random fluctuation. When we aggregate multiple data points within a time window(compute the mean of median), random noise tends to cancel out, revealing the underlying signal.
- Alignment: Create a common temporal reference frame.Different sensors record at different timestamps.
- Comparability:Standardized structure anables statistical comparsion.

### Baseline Choosing
- Subjective Recording: A physiologically stable condition without acute stress or strong activation.
- Shape in the plot: A plateau phase exhibiting minimal variance and no discernible trend.
- Low HR + High HRV

<img width="1104" height="756" alt="image" src="https://github.com/user-attachments/assets/ab01d65d-a5ff-4c2f-abda-f42bf981966b" />

### Code Learning

```python
#-------------(Add the showing of the target subjective recording labels)------------
record_labels=[
    ("2026-02-16 17:54","2026-02-16 19:45","Tolerable Social Context","#FF8C00"),
    ("2026-02-17 09:49","2026-02-17 14:14","Poor Awaking, Recovery","#87CEEB"),
    ("2026-02-17 15:13","2026-02-17 19:18","Extreme Burnout(Mental&Physical)","DarkRed"),
    ("2026-02-18 08:08","2026-02-18 15:34","Suboptimal Recovery(Fatigue)","#87CEEB"),
    ("2026-02-18 16:20","2026-02-18 20:50","Extreme Burnout(Mental&Physical)","DarkRed"),
    ("2026-02-19 17:54","2026-02-19 22:50","Baseline","LightGray"),
    ("2026-02-20 14:38","2026-02-20 20:26","High-stress Social Context","#FF8C00"),
    ("2026-02-21 10:57","2026-02-21 15:00","High-stress Social Context","#FF8C00"),
    ("2026-02-21 17:10","2026-02-21 21:00","Enriching Social Context","#DDA0DD")
]
#Define a list that contains the StartDate, EndDate,Text,Color

 for start,end,text,col in record_labels:
        axes[i].axvspan(pd.to_datetime(start),pd.to_datetime(end),color=col,alpha=0.15)
        if axes[i]==axes[0]:
            axes[i].text(pd.to_datetime(start),axes[i].get_ylim()[1],text,fontsize=10,rotation=45,verticalalignment='bottom')

#-----------------scatterplot->lineplot--------------------------
sns.lineplot(data=data, x='startDate', y='value', ax=axes[i], marker='o', markersize=4, linewidth=1, alpha=0.8)
```

## 2026-02-28 — Data checking (Feb 27)-Verify the "Mindful App" manual-trigger method

- Export the data in that night to see if the mindful app method works

Problem:Failed to record the data needed due to the restrictive data import settings in the collection process

<img width="1186" height="1990" alt="download" src="https://github.com/user-attachments/assets/59551607-cea6-415c-a679-11a341506b29" />

## 2026-02-28 — Critical Chanllenge:Data Sparsity During Target Events

#### Problem Definition
- Signal Attrition: Failed to capture continuous HRV data due to the strict signal-quality thresholds of the PPG sensor. The Apple Watch requires an exceptional high-quality signal(minimal motion artifact) to compute HRV, leading to significant gaps.
- Trigger vs. Stream: The Mindful App serves only as a high-frequenct measurement trigger rather than a guarantee of continuous data streaming.
- Temporal Dynamics: HRV is a highly dynamic metric with fluctuations occuring on a minute-by-minute scale. The current 3-4 hour sampling interval is insufficient to capture these rapid physiological shifts.
- The suject is high intra-individual variability.

#### Proposed Solution 1: Deploying Polar H10 for High-Fidelity ECG Data
- Disaventages: High frction for long-term wear. Significant additional investment(¥700) adn requires high reliability throughout the entire training and real-time inference phases.
#### Proposed Solution 2: Synthetic HRV Inference via Continuous HR Data
- Disaventages: Theoretically, it is impossible to perfectly reconstruct HRV from HR alone, as HR represents the Mean of the heart rate, whereas HRV represents its Variance.
- Literature Review: Initial searches for direct modeling were unsuccessful, but two foundational papers provide a theoretical basis for Physiological Coupling.

J. Warren et al., "Predicting Heart Rate Variability from Heart Rate and Step Count for University Student Weekdays," 2024 46th Annual International Conference of the IEEE Engineering in Medicine and Biology Society (EMBC), Orlando, FL, USA, 2024, pp. 1-4, doi: 10.1109/EMBC53108.2024.10781904. keywords: {Performance evaluation;Sensitivity;Anxiety disorders;Machine learning;Data models;Heart rate variability;Wearable devices;Engineering in medicine and biology;Monitoring;Stress measurement;low cost;machine learning;stress;ubiquitous monitoring;wearable},

J. Sacha, “Interaction between heart rate and heart rate variability,” Ann. Noninvasive Electrocardiol., vol. 19, no. 3, pp. 207–216, March 2014.

#### Proposed Solution 3（Chosen）: Multi-Stage Virtual Sensing & Multimodal Fusion with Data Augmentation
- Physical OptimizationL:
Turning the watch to the inside of the wrist.
AFib History Activation(Force the watch into a highfrequency sampling mode. Few hours -> every 15 minutes)
- Stage 1: Virtual HRV Synthesis: Develop a "Proxy Model" to infer a Predicted HRV by leveraging the non-linear coupling between HR and HRV.
  (still needed as the state could change in 15 minutues)
- Stage 2: Multimodal State Classification: Integrate the synthetic HRV(Pred) with high-fidenlity contextual variables to build a robust classification model for the 7 target states.

#### Next steps:
1.Initial Feasibility Check：Export the data on Feb 28th to check if turning the watch in the inside of the waist and improve the efficiency of collectiong data.

2.In the next two day: 
- Continuous Logging: Maintain a high-fidelity manual log in Memos, capturing the precise start/end times of daily activities and mapping them to the 7 target states.
- Instant Sampling: Actively initiate 30-second manual AFib/ECG recordings during critical "state-shift" moments (e.g., peak focus or sudden anxiety) to capture high-precision R-R interval data.
- Contextual Tracking: Ensure the App Privacy Report is enabled to archive background app usage and interaction frequency.
- 
3.Data Integrity Review & Prototype Modeling
- Gap Analysis: Consolidate all exported data (HR, Measured HRV, Steps, App Privacy Report) into a single dataframe to perform "Cross-Referencing" with Memo labels.
- Feasibility Validation: Run a baseline correlation check to ensure that the "Fidgeting" (micro-movements) and "Anxiety" states are visibly distinguishable in the data.
- Adjustment: Identify any missing variables or "silent periods" in data collection before launching the final 2-week intensive data sprint.



        





