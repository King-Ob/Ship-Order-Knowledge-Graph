# Entity Resolution Quality Metrics

## Overview
This document defines metrics for measuring the quality of entity resolution (ER) in the Ship Order Knowledge Graph. These metrics help assess how well we identify and link duplicate ships across multiple data sources.

**Reference**: Entity Resolution Evaluation (https://en.wikipedia.org/wiki/Record_linkage#Evaluation)

---

## Evaluation Dataset

### Labeled Sample

A labeled sample is required for evaluation. This consists of:
- **True Positives (TP)**: Correctly identified matches
- **False Positives (FP)**: Incorrectly identified matches (non-matches marked as matches)
- **False Negatives (FN)**: Missed matches (actual matches not identified)
- **True Negatives (TN)**: Correctly identified non-matches

### Sample Size

- **Minimum**: 50 ship pairs for meaningful statistics
- **Recommended**: 100-200 ship pairs
- **Distribution**: 
  - 30% obvious matches (same hull symbol)
  - 40% moderate matches (same name + class)
  - 30% difficult matches (fuzzy name, different sources)

---

## Core Metrics

### 1. Precision

**Definition**: Of all pairs identified as matches, what fraction are actually matches?

**Formula**:
```
Precision = TP / (TP + FP)
```

**Interpretation**:
- **High Precision (≥ 0.95)**: Very few false matches
- **Medium Precision (0.80-0.94)**: Some false matches, acceptable
- **Low Precision (< 0.80)**: Too many false matches, needs improvement

**Example**:
- Identified 100 matches
- 95 are correct matches (TP = 95)
- 5 are incorrect (FP = 5)
- Precision = 95 / (95 + 5) = 0.95 (95%)

### 2. Recall

**Definition**: Of all actual matches, what fraction did we identify?

**Formula**:
```
Recall = TP / (TP + FN)
```

**Interpretation**:
- **High Recall (≥ 0.95)**: Very few missed matches
- **Medium Recall (0.80-0.94)**: Some missed matches, acceptable
- **Low Recall (< 0.80)**: Too many missed matches, needs improvement

**Example**:
- 100 actual matches exist
- Identified 90 matches (TP = 90)
- Missed 10 matches (FN = 10)
- Recall = 90 / (90 + 10) = 0.90 (90%)

### 3. F1-Score

**Definition**: Harmonic mean of precision and recall, balancing both metrics.

**Formula**:
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

**Interpretation**:
- **High F1 (≥ 0.90)**: Excellent balance
- **Medium F1 (0.75-0.89)**: Good balance
- **Low F1 (< 0.75)**: Needs improvement

**Example**:
- Precision = 0.95
- Recall = 0.90
- F1 = 2 × (0.95 × 0.90) / (0.95 + 0.90) = 0.924

---

## Additional Metrics

### 4. Deduplication Rate

**Definition**: Percentage of duplicate records resolved.

**Formula**:
```
Deduplication Rate = (Duplicates Resolved / Total Records) × 100
```

**Example**:
- Total records: 200
- Duplicates resolved: 50
- Deduplication Rate = (50 / 200) × 100 = 25%

### 5. Link Coverage

**Definition**: Percentage of ships with cross-source links.

**Formula**:
```
Link Coverage = (Ships with Links / Total Ships) × 100
```

**Example**:
- Total ships: 100
- Ships with links: 75
- Link Coverage = (75 / 100) × 100 = 75%

### 6. Source Coverage

**Definition**: Average number of sources per ship.

**Formula**:
```
Source Coverage = (Total Source Links / Total Ships)
```

**Example**:
- 100 ships
- 150 source links (some ships have multiple sources)
- Source Coverage = 150 / 100 = 1.5 sources per ship

---

## Before/After Comparison

### Metrics to Track

1. **Before ER**:
   - Total records: 200
   - Unique ships (by blocking key): 120
   - Duplicates: 80
   - Average sources per ship: 1.0

2. **After ER**:
   - Golden records: 100
   - Duplicates resolved: 100
   - Links created: 150
   - Average sources per ship: 2.0

### Example Report

```
Entity Resolution Report
========================

BEFORE ER:
----------
Total Records:           200
Unique Ships (blocking): 120
Duplicate Records:       80
Source Coverage:         1.0 sources/ship

AFTER ER:
---------
Golden Records:          100
Duplicates Resolved:     100
Links Created:           150
Source Coverage:         2.0 sources/ship

QUALITY METRICS:
----------------
Precision:               0.95 (95%)
Recall:                  0.90 (90%)
F1-Score:                0.924
Deduplication Rate:      50.0%
Link Coverage:           75.0%

IMPROVEMENT:
------------
Records Reduced:         100 (50% reduction)
Source Coverage:         +100% (doubled)
```

---

## Evaluation Methodology

### Step 1: Create Labeled Sample

1. **Select Sample Pairs**:
   - Randomly select 100-200 ship pairs
   - Include obvious, moderate, and difficult cases

2. **Manual Labeling**:
   - Review each pair
   - Label as: Match, Non-Match, or Uncertain
   - Document reasoning

3. **Save Labels**:
   ```csv
   ship1_id,ship2_id,is_match,confidence,notes
   ship-CVN-68,ship-CVN-68-NVR,true,high,Exact hull symbol match
   ship-CVN-68,ship-CVN-71,false,high,Different hull symbols
   ship-DDG-51,ship-DDG-51-WDQS,true,high,Same hull symbol, different sources
   ```

### Step 2: Run ER Algorithm

1. **Execute ER**: Run entity resolution on full dataset
2. **Extract Predictions**: Get predicted matches for sample pairs
3. **Compare**: Compare predictions with labels

### Step 3: Calculate Metrics

```python
def calculate_metrics(labels, predictions):
    """
    Calculate precision, recall, and F1-score.
    
    Args:
        labels: List of (pair_id, is_match) tuples (ground truth)
        predictions: List of (pair_id, is_match) tuples (predictions)
    
    Returns:
        Dictionary with precision, recall, F1
    """
    # Create lookup dictionaries
    label_dict = {pair_id: is_match for pair_id, is_match in labels}
    pred_dict = {pair_id: is_match for pair_id, is_match in predictions}
    
    # Calculate TP, FP, FN
    TP = 0  # True Positive: predicted match, actual match
    FP = 0  # False Positive: predicted match, actual non-match
    FN = 0  # False Negative: predicted non-match, actual match
    
    all_pairs = set(label_dict.keys()) | set(pred_dict.keys())
    
    for pair_id in all_pairs:
        label_match = label_dict.get(pair_id, False)
        pred_match = pred_dict.get(pair_id, False)
        
        if pred_match and label_match:
            TP += 1
        elif pred_match and not label_match:
            FP += 1
        elif not pred_match and label_match:
            FN += 1
        # TN (True Negative) not needed for precision/recall
    
    # Calculate metrics
    precision = TP / (TP + FP) if (TP + FP) > 0 else 0.0
    recall = TP / (TP + FN) if (TP + FN) > 0 else 0.0
    f1 = 2 * (precision * recall) / (precision + recall) if (precision + recall) > 0 else 0.0
    
    return {
        'precision': precision,
        'recall': recall,
        'f1_score': f1,
        'true_positives': TP,
        'false_positives': FP,
        'false_negatives': FN
    }
```

### Step 4: Generate Report

```python
def generate_report(metrics, before_stats, after_stats):
    """Generate ER quality report."""
    report = f"""
Entity Resolution Quality Report
================================

BEFORE ER:
----------
Total Records:           {before_stats['total_records']}
Unique Ships:            {before_stats['unique_ships']}
Duplicate Records:       {before_stats['duplicates']}
Source Coverage:         {before_stats['source_coverage']:.1f} sources/ship

AFTER ER:
---------
Golden Records:          {after_stats['golden_records']}
Duplicates Resolved:    {after_stats['duplicates_resolved']}
Links Created:           {after_stats['links_created']}
Source Coverage:         {after_stats['source_coverage']:.1f} sources/ship

QUALITY METRICS:
----------------
Precision:               {metrics['precision']:.3f} ({metrics['precision']*100:.1f}%)
Recall:                  {metrics['recall']:.3f} ({metrics['recall']*100:.1f}%)
F1-Score:                {metrics['f1_score']:.3f}
True Positives:          {metrics['true_positives']}
False Positives:         {metrics['false_positives']}
False Negatives:         {metrics['false_negatives']}

IMPROVEMENT:
------------
Records Reduced:         {before_stats['total_records'] - after_stats['golden_records']} 
                         ({(1 - after_stats['golden_records']/before_stats['total_records'])*100:.1f}% reduction)
Source Coverage:         +{(after_stats['source_coverage']/before_stats['source_coverage'] - 1)*100:.0f}%
"""
    return report
```

---

## Quality Targets

### Minimum Acceptable

- **Precision**: ≥ 0.85 (85%)
- **Recall**: ≥ 0.80 (80%)
- **F1-Score**: ≥ 0.82

### Good Quality

- **Precision**: ≥ 0.90 (90%)
- **Recall**: ≥ 0.85 (85%)
- **F1-Score**: ≥ 0.87

### Excellent Quality

- **Precision**: ≥ 0.95 (95%)
- **Recall**: ≥ 0.90 (90%)
- **F1-Score**: ≥ 0.92

---

## Error Analysis

### Common Error Types

1. **False Positives (Incorrect Matches)**:
   - Different ships with similar names
   - Example: USS America (LHA-6) vs USS America (CV-66)
   - **Fix**: Require hull symbol match or additional verification

2. **False Negatives (Missed Matches)**:
   - Same ship with different name formats
   - Example: "USS Nimitz" vs "Nimitz"
   - **Fix**: Improve name normalization

3. **Blocking Key Issues**:
   - Missing identifiers causing poor blocking
   - **Fix**: Improve fallback blocking strategies

### Error Analysis Report

```python
def analyze_errors(labels, predictions):
    """Analyze types of errors."""
    errors = {
        'false_positives': [],
        'false_negatives': []
    }
    
    label_dict = {pair_id: is_match for pair_id, is_match in labels}
    pred_dict = {pair_id: is_match for pair_id, is_match in predictions}
    
    for pair_id in set(label_dict.keys()) | set(pred_dict.keys()):
        label_match = label_dict.get(pair_id, False)
        pred_match = pred_dict.get(pair_id, False)
        
        if pred_match and not label_match:
            errors['false_positives'].append(pair_id)
        elif not pred_match and label_match:
            errors['false_negatives'].append(pair_id)
    
    return errors
```

---

## Continuous Monitoring

### Regular Evaluation

1. **Monthly**: Evaluate on new data
2. **Quarterly**: Full evaluation on labeled sample
3. **After Changes**: Evaluate after ER algorithm updates

### Metrics Dashboard

Track over time:
- Precision trend
- Recall trend
- F1-Score trend
- Deduplication rate
- Link coverage

---

## References

- **Entity Resolution**: https://en.wikipedia.org/wiki/Record_linkage
- **Precision and Recall**: https://en.wikipedia.org/wiki/Precision_and_recall
- **F1-Score**: https://en.wikipedia.org/wiki/F-score
- **OpenRefine**: https://openrefine.org/

---

## Change Log

- **2024-01-15**: Initial ER metrics documentation created
  - Defined precision, recall, F1-score
  - Created evaluation methodology
  - Added quality targets
  - Created error analysis framework
