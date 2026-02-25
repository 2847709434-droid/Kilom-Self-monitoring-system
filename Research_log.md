# Research Log

## 2026-02-26-4:09-2026-0216-0219 Exploration
### Code learing 
import matplotlib.date as mdates ---横坐标显示细化到小时
import seaborn as sns ---accurate diagrams based on matplot
path=r"..." ---r:Raw String: refer this string as normal character \0
tree=ET.parse(path)
root=tree.getroot() ---XML文件提取
df=pd.DataFrame([list.attrib for list in root.findall('Record')]) ---turn XML into Dataframe
(Background:苹果健康数据中，几乎每一条心率，步数，睡眠记录都是以<Record.../>方式存在）
### 电脑操作
shift+ctrl+C ---documents' address fast copying
