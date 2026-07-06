# Matplotlib Data Visualization Reference Guide

This comprehensive guide covers 7 essential types of data visualizations using Python's Matplotlib library. Each example builds upon fundamental concepts to help you create professional-quality graphs.

## Example 1: Line Plot
**Purpose**: Shows trends and changes over time or continuous data.

```python
import matplotlib.pyplot as plt

# Data preparation
months = ["Jan","Feb","Mar","Apr","May","Jun"]
sales = [120,150,180,170,210,250]

# Create figure with specific size
plt.figure(figsize=(10,6))

# Create line plot with customizations
plt.plot(
         months,
         sales,
         color='blue',           # Line color
         linewidth=3,            # Line thickness
         linestyle='--',         # Line style (dashed)
         
         marker='o',             # Marker shape
         markersize=10,          # Marker size
         
         markerfacecolor='yellow',    # Marker fill color
         markeredgecolor='red',       # Marker border color
         markeredgewidth=2,           # Marker border thickness
         
         label='Months Sales'    # Legend label
        )

# Add titles and labels
plt.title('Monthly Sales Data', fontsize=16, fontweight='bold')
plt.xlabel('Months', fontsize=14)
plt.ylabel('Sales', fontsize=14)

# Add grid for better readability
plt.grid(True)

# Set axis limits
plt.xlim("Jan","Jun")
plt.ylim(100,300)

# Add legend
plt.legend()

# Add annotation to highlight specific point
plt.annotate("Highest Sales",
             xy=('Jun',250),           # Point to annotate
             xytext=('Apr', 270),      # Text position
             arrowprops=dict(facecolor='black', shrink=0.01))

# Save the plot
plt.savefig("vpro1.png")

# Display the plot
plt.show()
```

**Key Learning Points**:
- Line plots are ideal for time series data
- Markers help identify individual data points
- Annotations can highlight important insights
- Grid improves data readability

---

## Example 2: Bar Chart
**Purpose**: Compares discrete categories or groups of data.

```python
import matplotlib.pyplot as plt

# Data preparation
months = ["Jan","Feb","Mar","Apr","May","Jun"]
sales = [100,120,140,180,220,300]

# Create figure
plt.figure(figsize=(10,6))

# Create bar chart
bars = plt.bar(months, sales, 
               color='skyblue',      # Bar color
               edgecolor='black',    # Bar border color
               linewidth=2,          # Border thickness
               width=0.6,            # Bar width
               label='Sales')        # Legend label

# Add value labels on top of each bar
for bar in bars:
    plt.text(bar.get_x() + bar.get_width()/2,    # X position (center of bar)
             bar.get_height() + 5,                # Y position (above bar)
             bar.get_height(),                    # Text content
             ha='center',                         # Horizontal alignment
             fontsize=10,
             fontweight='bold',
             color='red')

# Add labels and title
plt.xlabel("Months", fontsize=12)
plt.ylabel("Sales", fontsize=12) 
plt.title("Months and Sales Data")

# Add grid (only horizontal lines)
plt.grid(axis='y', linestyle='--', alpha=0.7)

# Set axis limits
plt.xlim(-0.5, 5.5)
plt.ylim(50, 350)

# Add annotation for highest value
plt.annotate("Highest Sales",
             xy=(5,300),                          # Point to annotate
             xytext=(4,310),                      # Text position
             arrowprops=dict(facecolor='red', shrink=0.05),
             fontsize=11,
             color='red')

# Add legend and save
plt.legend()
plt.savefig("vpro2.png")
plt.show()
```

**Key Learning Points**:
- Bar charts excel at comparing categories
- Value labels make exact values clear
- Grid lines help estimate values
- Annotations can highlight key insights

---

## Example 3: Pie Chart
**Purpose**: Shows proportions and percentages of a whole.

```python
import matplotlib.pyplot as plt

# Data preparation
subjects = ["Python","Java","React","SQL","AWS"]
marks = [95,85,70,60,90]
colors = ['gold','skyblue','lightgreen','orange','pink']
explode = (0.1,0,0,0,0)  # Explode first slice (Python)

# Create figure
plt.figure(figsize=(8,8))

# Create pie chart
plt.pie(
    marks,
    labels=subjects,
    colors=colors,
    explode=explode,          # Separate slices
    autopct='%1.1f%%',        # Show percentages
    startangle=90,            # Rotate chart
    shadow=True,              # Add shadow effect
    counterclock=True,        # Direction of slices
    radius=0.9,               # Chart size
    pctdistance=0.7,          # Distance of percentage labels
    labeldistance=1.1,        # Distance of category labels
    wedgeprops={              # Slice properties
        'edgecolor':'black',
        'linewidth' : 2
    },
    textprops={               # Text properties
        'fontsize':12,
        'color':'black'
    }
)

# Add title and legend
plt.title("Student Marks Distribution", fontsize=18, fontweight='bold')
plt.legend(title="Subjects", loc="upper right")

# Save and display
plt.savefig("vpro3.png")
plt.show()
```

**Key Learning Points**:
- Pie charts show parts of a whole
- Exploding slices emphasizes important categories
- Percentages provide exact proportional information
- Colors help distinguish categories

---

## Example 4: Scatter Plot
**Purpose**: Shows relationships between two continuous variables.

```python
import matplotlib.pyplot as plt

# Data preparation
study_hours = [1,2,3,4,5,6,7,8]
marks = [35,42,50,60,68,75,88,95]
sizes = [80,100,120,140,160,180,200,220]  # Point sizes
colors = ['red','green','blue','orange','purple','brown','pink','cyan']

# Create figure
plt.figure(figsize=(10,6))

# Create scatter plot
plt.scatter(study_hours,
            marks,
            s=sizes,                    # Point sizes
            c=colors,                   # Point colors
            marker='o',                 # Point shape
            alpha=0.8,                  # Transparency
            edgecolors='black',         # Point borders
            linewidths=2,               # Border thickness
            label="Students")

# Add titles and labels
plt.title("Study Hours Vs Marks", fontsize=18, color="red", fontweight="bold")
plt.xlabel("Study Hours", fontsize=12, color="red")
plt.ylabel("Marks", fontsize=12, color="red")

# Add grid and set limits
plt.grid(True, linestyle='--', alpha=0.7)
plt.xlim(0,9)
plt.ylim(30,100)

# Add annotation
plt.annotate("Top Student",
             xy=(8,95),                 # Point to annotate
             xytext=(6.5,90),           # Text position
             arrowprops=dict(facecolor='red', shrink=0.05))

# Add legend and save
plt.legend()
plt.savefig("vpro4.png")
plt.show()
```

**Key Learning Points**:
- Scatter plots reveal correlations between variables
- Point size can represent a third dimension of data
- Colors can categorize different groups
- Annotations help identify outliers or important points

---

## Example 5: Histogram
**Purpose**: Shows the distribution and frequency of continuous data.

```python
import matplotlib.pyplot as plt

# Data preparation
marks = [35,40,42,45,48,
         50,52,55,58,60,
         62,65,68,70,72,
         75,78,80,82,85,
         88,90,92,95]

# Understanding bins calculation:
# bins = 6
# Range: 95 - 35 = 60
# Bin width: 60 / 6 = 10
# Bins: 35-45, 45-55, 55-65, 65-75, 75-85, 85-95

# Create figure
plt.figure(figsize=(10,6))

# Create histogram
plt.hist(marks,
         bins=6,                        # Number of bins
         color='skyblue',               # Bar color
         edgecolor='black',             # Bar border color
         linewidth=2,                   # Border thickness
         alpha=0.8,                     # Transparency
         histtype='stepfilled',         # Histogram type
         rwidth=0.9,                    # Bar width ratio
         label='Students')

# Add titles and labels
plt.title("Student Marks Distribution", fontsize=18, fontweight='bold')
plt.xlabel("Marks", fontsize=12)
plt.ylabel("Number of Students", fontsize=12)

# Add grid and set limits
plt.grid(axis='y', linestyle='--', alpha=0.5)
plt.xlim(30,100)
plt.ylim(0,6)

# Add legend and save
plt.legend()
plt.savefig("vpro5.png")
plt.show()
```

**Key Learning Points**:
- Histograms show data distribution patterns
- Bin size affects the visualization granularity
- Frequency on y-axis shows how often values occur
- Useful for identifying normal distributions, skewness, and outliers

---

## Example 6: Subplots
**Purpose**: Display multiple related charts in a single figure for comparison.

```python
import matplotlib.pyplot as plt

# Data preparation
subjects = ["Python","Java","React","SQL"]
marks = [95,85,75,90]

# Create figure with subplots
plt.figure(figsize=(12,8))

# Subplot 1: Line Plot
plt.subplot(2,2,1)                     # 2 rows, 2 columns, position 1
plt.plot(subjects, marks, marker='o')
plt.title("Line Plot")

# Subplot 2: Bar Chart
plt.subplot(2,2,2)                     # 2 rows, 2 columns, position 2
plt.bar(subjects, marks, color='orange')
plt.title("Bar Chart")

# Subplot 3: Pie Chart
plt.subplot(2,2,3)                     # 2 rows, 2 columns, position 3
plt.pie(marks, labels=subjects, autopct='%1.1f%%')
plt.title("Pie Chart")

# Subplot 4: Histogram
plt.subplot(2,2,4)                     # 2 rows, 2 columns, position 4
plt.hist(marks, bins=4, color='green', edgecolor='black')  
plt.title("Histogram")

# Adjust layout to prevent overlap
plt.tight_layout()

# Save and display
plt.savefig("vpro6.png")
plt.show()
```

**Key Learning Points**:
- Subplots allow multiple visualizations in one figure
- `plt.subplot(rows, columns, position)` creates grid layout
- `tight_layout()` automatically adjusts spacing
- Different chart types can show different aspects of the same data

---

## Example 7: Box Plot
**Purpose**: Shows data distribution, quartiles, median, and outliers.

```python
import matplotlib.pyplot as plt

# Data preparation (including an outlier)
marks = [35,40,45,50,55,
         60,65,70,75,80,
         85,90,95,98,150]  # 150 is an outlier

# Create figure
plt.figure(figsize=(8,6))

# Create box plot
plt.boxplot(marks,
            notch=True,                 # Add notch around median
            vert=True,                  # Vertical orientation
            patch_artist=True,          # Enable color filling
            widths=0.5,                 # Box width
            showmeans=False,            # Don't show mean
            showfliers=False,           # Don't show outliers
            labels=['Students'],        # Box label
            
            # Box styling
            boxprops=dict(facecolor='skyblue', color='blue', linewidth=2),
            
            # Median line styling
            medianprops=dict(color='red', linewidth=3),
            
            # Whisker styling
            whiskerprops=dict(color='green', linewidth=2),
            
            # Cap styling
            capprops=dict(color='orange', linewidth=2),
            
            # Outlier styling
            flierprops=dict(marker='o', markerfacecolor='red', markersize=10)
            )

plt.show()
```

**Key Learning Points**:
- Box plots show five-number summary (min, Q1, median, Q3, max)
- Outliers are displayed as separate points
- Notches around median show confidence intervals
- Whiskers extend to show data range
- Useful for comparing distributions across groups

---

## Summary

This reference guide covers seven fundamental visualization types:

1. **Line Plot**: Trends over time
2. **Bar Chart**: Category comparisons
3. **Pie Chart**: Part-to-whole relationships
4. **Scatter Plot**: Variable correlations
5. **Histogram**: Data distributions
6. **Subplots**: Multiple chart comparisons
7. **Box Plot**: Statistical summaries

Each visualization type serves specific analytical purposes and can be customized extensively using Matplotlib's rich parameter options.
