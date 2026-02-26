# Research Log

---

## 2026-02-26 — Physio Data Exploration (Feb 16–19)

### Objective

Explore physiological signals collected during family gatherings and outings.

---

### Code Learning

```python
import matplotlib.dates as mdates  # format x-axis to hour level
import seaborn as sns              # statistical visualization based on matplotlib

path = r"..."
tree = ET.parse(path)
root = tree.getroot()

df = pd.DataFrame(
    [list(attrib.values()) for attrib in root.findall('Record')]
)
