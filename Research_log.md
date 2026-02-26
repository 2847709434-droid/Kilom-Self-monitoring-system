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
HR:irregular and context-dependent sampling frequency
Respiratory Rate:Recorded sporadically under resting conditions
###### Solutions 2:Manual Forced Recording via Zhengnian App
When entering a relevent state:
    Manually open Zhengnian App
    Remain physically steady for 1-5 minutes
    Ignore initial transitional data
    Maintain natural physiological rhythm
Rationale:Physiology changes are continuous rather than abrupt,A 1-5 minute steady recording window introduces accepteble error.
Forced recording offer "Valuable states"labels.

### Additional Vairable:Wearing earpods


