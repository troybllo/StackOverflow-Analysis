# StackOverflow Data Analysis with Distributed K-Means Clustering

A Scala-based distributed computing project that analyzes StackOverflow posts using Apache Spark and implements a parallel K-means clustering algorithm to understand answer score distributions across different programming language communities.

## Overview

This project clusters StackOverflow questions and answers by their scores to analyze documentation quality patterns across programming languages. The goal is to understand how answer ratings vary between different language communities and identify potential gaps in documentation availability.

## Key Features

- **Distributed Processing**: Uses Apache Spark RDDs for parallel data processing
- **K-Means Clustering**: Custom implementation of the K-means algorithm for large-scale data
- **Multi-Language Analysis**: Compares answer quality patterns across 15 programming languages
- **Score-Based Clustering**: Groups posts by answer scores to identify documentation patterns

## Supported Languages

JavaScript, Java, PHP, Python, C#, C++, Ruby, CSS, Objective-C, Perl, Scala, Haskell, MATLAB, Clojure, Groovy

## Data Processing Pipeline

### 1. Data Parsing (`rawPostings`)
- Parses CSV data containing StackOverflow posts
- Handles questions (postTypeId=1) and answers (postTypeId=2)
- Extracts post metadata including scores, tags, and relationships

### 2. Question-Answer Grouping (`groupedPostings`)
- **Algorithm**: RDD join operations
- Groups answers with their parent questions using post IDs
- Creates `RDD[(QuestionID, Iterable[(Question, Answer)])]`
- Uses efficient key-based joins to avoid expensive shuffling

### 3. Score Computation (`scoredPostings`)
- Finds the highest-scored answer for each question
- Implements linear scan algorithm for maximum score detection
- Returns `RDD[(Question, HighestScore)]` pairs

### 4. Vector Creation (`vectorPostings`)
- **Feature Engineering**: Converts posts to clustering vectors
- Creates 2D vectors: `(language_index × langSpread, highest_score)`
- Uses `langSpread=50000` to ensure language separation in clustering space
- Filters out posts without recognizable programming language tags

### 5. K-Means Clustering (`kmeans`)
- **Algorithm**: Iterative centroid-based clustering
- **Distance Metric**: Euclidean distance
- **Convergence**: Stops when centroid movement < threshold (20.0)
- **Parallelization**: Distributed centroid assignment and averaging
- Typically converges in 24-54 iterations depending on language spread

### 6. Results Analysis (`clusterResults`)
- Computes cluster statistics:
  - **Dominant Language**: Most frequent language in each cluster
  - **Language Percentage**: Ratio of dominant language posts
  - **Cluster Size**: Total number of questions
  - **Median Score**: Middle value of answer scores

## Technical Implementation

### Core Algorithms
- **Distributed Join**: Efficient question-answer pairing using Spark joins
- **K-Means**: Custom parallel implementation with configurable parameters
- **Vector Space Mapping**: Language-score coordinate system for clustering

### Key Parameters
- `langSpread = 50000`: Distance multiplier between languages
- `kmeansKernels = 45`: Number of cluster centroids (3 per language)
- `kmeansEta = 20.0`: Convergence threshold
- `kmeansMaxIterations = 120`: Maximum clustering iterations

### Performance Optimizations
- Uses `flatMap` to filter invalid data points early
- Leverages Spark's `groupByKey` and `collectAsMap` for efficient aggregations
- Implements reservoir sampling for representative data subsets

## Usage

```bash
# Compile the project
sbt compile

# Run the analysis
sbt run

# Run tests
sbt test
```

## Output

The program outputs clustering results showing:
```
Score  Dominant language (%percent)  Questions
================================================
    1  C#                (100.0%)      17442
   34  C#                (100.0%)        364
  173  C#                (100.0%)         49
    2  C++               (100.0%)       8849
```

## Scientific Relevance

This analysis helps identify:
- **Documentation Gaps**: Languages with fewer high-scoring answers may lack quality documentation
- **Community Engagement**: Score distributions reflect community activity and expertise
- **Knowledge Patterns**: Clustering reveals how different languages handle similar programming challenges

The distributed approach enables analysis of large-scale StackOverflow datasets that would be computationally prohibitive with traditional single-machine approaches.
