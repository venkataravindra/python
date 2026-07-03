# 📊 **Matplotlib Reference Guide - Complete Visualization Toolkit**

## 🎯 **Basic Setup and Imports**

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
```

---

## 📈 **Line Plots**

### **Basic Line Plot**

```python
# 1. Simple Line Plot
plt.plot(x, y, color='blue', linewidth=2, linestyle='-', marker='o', label='Data')
```

**Parameters:**
- `color`: Line color ('blue', 'red', '#FF5733', etc.)
- `linewidth`: Thickness of the line
- `linestyle`: '-' (solid), '--' (dashed), '-.' (dash-dot), ':' (dotted)
- `marker`: 'o' (circle), 's' (square), '^' (triangle), '*' (star), etc.
- `label`: Legend label

### **Advanced Line Plot Example**

```python
# Complete Line Plot with All Features
months = ["Jan","Feb","Mar","Apr","May","Jun"]
sales = [120,150,180,170,210,250]

plt.figure(figsize=(10,6))

plt.plot(
    months,
    sales,
    color='blue',           # Line color
    linewidth=3,            # Line thickness
    linestyle='--',         # Dashed line
    
    marker='o',             # Circle markers
    markersize=10,          # Marker size
    markerfacecolor='yellow',   # Marker fill color
    markeredgecolor='red',      # Marker border color
    markeredgewidth=2,          # Marker border width
    
    label='Monthly Sales'   # Legend label
)

# Customization
plt.title('Monthly Sales Data', fontsize=16, fontweight='bold')
plt.xlabel('Months', fontsize=14)
plt.ylabel('Sales', fontsize=14)
plt.grid(True)
plt.xlim("Jan","Jun")
plt.ylim(100,300)
plt.legend()

# Add annotation
plt.annotate("Highest Sales",
             xy=('Jun',250),        # Point to annotate
             xytext=('Apr', 270),   # Text position
             arrowprops=dict(facecolor='black', shrink=0.01))

plt.savefig("line_plot.png", dpi=300, bbox_inches='tight')
plt.show()
```

---

## 📊 **Bar Charts**

### **Basic Bar Chart**

```python
# 2. Bar Chart
plt.bar(x, height, width=0.8, color='skyblue', edgecolor='black', linewidth=1)
```

**Parameters:**
- `x`: X-axis positions
- `height`: Bar heights
- `width`: Bar width (0.0 to 1.0)
- `color`: Bar fill color
- `edgecolor`: Bar border color
- `linewidth`: Border thickness

### **Advanced Bar Chart Example**

```python
# Complete Bar Chart with All Features
months = ["Jan","Feb","Mar","Apr","May","Jun"]
sales = [100,120,140,180,220,300]

plt.figure(figsize=(10,6))

bars = plt.bar(months, sales, 
               color='skyblue',      # Bar color
               edgecolor='black',    # Border color
               linewidth=2,          # Border width
               width=0.6,            # Bar width
               label='Sales')        # Legend label

# Add value labels on bars
for bar in bars:
    plt.text(bar.get_x() + bar.get_width()/2,   # X position (center)
             bar.get_height() + 5,               # Y position (above bar)
             bar.get_height(),                   # Text content
             ha='center',                        # Horizontal alignment
             fontsize=10,
             fontweight='bold',
             color='red')

# Customization
plt.xlabel("Months", fontsize=12)
plt.ylabel("Sales", fontsize=12) 
plt.title("Monthly Sales Data", fontsize=14, fontweight='bold')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.xlim(-0.5, 5.5)
plt.ylim(0, 350)

# Add annotation
plt.annotate("Highest Sales",
             xy=(5,300),                        # Point to annotate
             xytext=(4,320),                    # Text position
             arrowprops=dict(facecolor='red', shrink=0.05),
             fontsize=11,
             color='red')

plt.legend()
plt.savefig("bar_chart.png", dpi=300, bbox_inches='tight')
plt.show()
```

### **Horizontal Bar Chart**

```python
# 3. Horizontal Bar Chart
plt.barh(y, width, height=0.8, color='lightgreen', edgecolor='black')
```

**Example:**
```python
categories = ['A', 'B', 'C', 'D']
values = [23, 45, 56, 78]

plt.figure(figsize=(8, 6))
plt.barh(categories, values, color='lightgreen', edgecolor='black')
plt.xlabel('Values')
plt.ylabel('Categories')
plt.title('Horizontal Bar Chart')
plt.show()
```

---

## 🥧 **Pie Charts**

### **Basic Pie Chart**

```python
# 4. Pie Chart
plt.pie(sizes, labels=None, colors=None, autopct=None, startangle=0, explode=None)
```

**Parameters:**
- `sizes`: Wedge sizes
- `labels`: Wedge labels
- `colors`: Wedge colors
- `autopct`: Format string for percentage labels
- `startangle`: Starting angle for first wedge
- `explode`: Fraction to separate wedges

### **Advanced Pie Chart Example**

```python
# Complete Pie Chart with All Features
categories = ['Product A', 'Product B', 'Product C', 'Product D']
sales = [30, 25, 20, 25]
colors = ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99']
explode = (0.1, 0, 0, 0)  # Explode first slice

plt.figure(figsize=(10, 8))

wedges, texts, autotexts = plt.pie(
    sales,
    labels=categories,
    colors=colors,
    autopct='%1.1f%%',      # Percentage format
    startangle=90,          # Start angle
    explode=explode,        # Explode slices
    shadow=True,            # Add shadow
    textprops={'fontsize': 12}
)

# Customize text
for autotext in autotexts:
    autotext.set_color('white')
    autotext.set_fontweight('bold')

plt.title('Sales Distribution by Product', fontsize=16, fontweight='bold', pad=20)
plt.axis('equal')  # Equal aspect ratio ensures circular pie

# Add legend
plt.legend(wedges, categories, title="Products", loc="center left", bbox_to_anchor=(1, 0, 0.5, 1))

plt.savefig("pie_chart.png", dpi=300, bbox_inches='tight')
plt.show()
```

---

## 📈 **Advanced Plot Types**

### **Scatter Plot**

```python
# 5. Scatter Plot
plt.scatter(x, y, s=None, c=None, marker='o', alpha=None, edgecolors=None)
```

**Example:**
```python
x = np.random.randn(100)
y = np.random.randn(100)
colors = np.random.rand(100)

plt.figure(figsize=(8, 6))
plt.scatter(x, y, 
           s=50,                    # Size
           c=colors,                # Color array
           alpha=0.7,               # Transparency
           edgecolors='black',      # Edge color
           linewidth=0.5)           # Edge width

plt.colorbar()  # Add colorbar
plt.xlabel('X values')
plt.ylabel('Y values')
plt.title('Scatter Plot')
plt.grid(True, alpha=0.3)
plt.show()
```

### **Histogram**

```python
# 6. Histogram
plt.hist(x, bins=10, color='skyblue', edgecolor='black', alpha=0.7, density=False)
```

**Example:**
```python
data = np.random.normal(100, 15, 1000)

plt.figure(figsize=(10, 6))
n, bins, patches = plt.hist(data, 
                           bins=30,
                           color='skyblue',
                           edgecolor='black',
                           alpha=0.7,
                           density=True)  # Normalize to show probability density

plt.xlabel('Values')
plt.ylabel('Frequency')
plt.title('Histogram Example')
plt.grid(True, alpha=0.3)
plt.show()
```

### **Box Plot**

```python
# 7. Box Plot
plt.boxplot(x, labels=None, notch=False, patch_artist=False)
```

**Example:**
```python
data = [np.random.normal(0, std, 100) for std in range(1, 4)]

plt.figure(figsize=(8, 6))
box_plot = plt.boxplot(data, 
                      labels=['Group 1', 'Group 2', 'Group 3'],
                      patch_artist=True,
                      notch=True)

# Customize colors
colors = ['lightblue', 'lightgreen', 'lightcoral']
for patch, color in zip(box_plot['boxes'], colors):
    patch.set_facecolor(color)

plt.ylabel('Values')
plt.title('Box Plot Example')
plt.grid(True, alpha=0.3)
plt.show()
```

---

## 🎨 **Customization and Styling**

### **Figure and Subplot Management**

```python
# 8. Figure creation and management
plt.figure(figsize=(width, height), dpi=100, facecolor='white', edgecolor='black')
```

**Example:**
```python
# Create figure with subplots
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Plot in each subplot
axes[0, 0].plot([1, 2, 3], [1, 4, 2])
axes[0, 0].set_title('Plot 1')

axes[0, 1].bar(['A', 'B', 'C'], [1, 3, 2])
axes[0, 1].set_title('Plot 2')

axes[1, 0].scatter([1, 2, 3], [2, 3, 1])
axes[1, 0].set_title('Plot 3')

axes[1, 1].pie([1, 2, 3], labels=['A', 'B', 'C'])
axes[1, 1].set_title('Plot 4')

plt.tight_layout()  # Adjust spacing
plt.show()
```

### **Text and Annotations**

```python
# 9. Text and annotations
plt.text(x, y, s, fontsize=12, color='black', ha='center', va='center')
plt.annotate(text, xy=(x, y), xytext=(x, y), arrowprops=dict())
```

**Text Parameters:**
- `ha`: Horizontal alignment ('left', 'center', 'right')
- `va`: Vertical alignment ('bottom', 'center', 'top')
- `fontsize`: Text size
- `fontweight`: 'normal', 'bold', 'light', 'heavy'
- `color`: Text color

**Annotation Parameters:**
- `xy`: Point to annotate
- `xytext`: Text position
- `arrowprops`: Arrow properties dictionary

### **Grid and Axes Customization**

```python
# 10. Grid customization
plt.grid(visible=True, which='major', axis='both', color='gray', linestyle='-', linewidth=0.5, alpha=0.5)
```

**Grid Parameters:**
- `which`: 'major', 'minor', 'both'
- `axis`: 'x', 'y', 'both'
- `linestyle`: '-', '--', '-.', ':'
- `alpha`: Transparency (0-1)

```python
# 11. Axes limits and ticks
plt.xlim(left, right)
plt.ylim(bottom, top)
plt.xticks(ticks, labels, rotation=0)
plt.yticks(ticks, labels)
```

**Example:**
```python
x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y)

# Customize axes
plt.xlim(0, 10)
plt.ylim(-1.5, 1.5)
plt.xticks(np.arange(0, 11, 2), ['0', '2π', '4π', '6π', '8π', '10π'])
plt.yticks([-1, 0, 1], ['Min', 'Zero', 'Max'])

# Grid
plt.grid(True, linestyle='--', alpha=0.7)

plt.xlabel('X values')
plt.ylabel('Y values')
plt.title('Sine Wave')
plt.show()
```

### **Colors and Styles**

```python
# 12. Color specifications
# Named colors: 'red', 'blue', 'green', 'orange', 'purple', 'brown', 'pink', 'gray', 'olive', 'cyan'
# Hex colors: '#FF5733', '#33FF57', '#3357FF'
# RGB tuples: (0.1, 0.2, 0.5)
# Grayscale: '0.75' (75% gray)
```

```python
# 13. Style sheets
plt.style.use('seaborn')  # Available styles: 'default', 'classic', 'seaborn', 'ggplot', 'bmh', 'dark_background'
```

### **Legends**

```python
# 14. Legend customization
plt.legend(labels=None, loc='best', fontsize='medium', title=None, frameon=True, shadow=False)
```

**Legend Locations:**
- 'best', 'upper right', 'upper left', 'lower left', 'lower right'
- 'right', 'center left', 'center right', 'lower center', 'upper center', 'center'

**Example:**
```python
x = np.linspace(0, 10, 100)

plt.figure(figsize=(10, 6))
plt.plot(x, np.sin(x), label='sin(x)', linewidth=2)
plt.plot(x, np.cos(x), label='cos(x)', linewidth=2)
plt.plot(x, np.tan(x), label='tan(x)', linewidth=2)

plt.legend(loc='upper right', 
          fontsize=12, 
          title='Functions',
          frameon=True,
          shadow=True,
          fancybox=True)

plt.ylim(-2, 2)
plt.grid(True, alpha=0.3)
plt.show()
```

---

## 💾 **Saving and Export**

### **Save Figure**

```python
# 15. Save figure
plt.savefig(fname, dpi=None, facecolor='w', edgecolor='w', orientation='portrait', 
           papertype=None, format=None, transparent=False, bbox_inches=None, 
           pad_inches=0.1, metadata=None)
```

**Save Parameters:**
- `fname`: Filename with extension (.png, .pdf, .svg, .eps, .jpg)
- `dpi`: Resolution in dots per inch
- `bbox_inches`: 'tight' to remove extra whitespace
- `transparent`: Transparent background
- `format`: File format ('png', 'pdf', 'svg', 'eps', 'jpg')

**Example:**
```python
plt.figure(figsize=(10, 6))
plt.plot([1, 2, 
