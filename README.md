# Deep Image Retrieval

## Introduction
The goal of this project is deep image retrieval, that is learning an embedding (or mapping) from images to a compact latent space in which cosine similarity between two learned embeddings correspond to a ranking measure for image retrieval task.

## Data

We used two popular Image retrieval datasets published by the Oxford Visual Geometry Group for this project,
1.  Oxford Buildings dataset
2.  Paris Buildings dataset

Both datasets consists of images collected from Flickr by searching for particular landmarks. The collection has been manually annotated to generate a comprehensive ground truth for all 11 different landmarks per dataset, each represented by 5 queries. This gives a set of 55 queries over which an object retrieval system can be evaluated.

We used an 80/ 20 ratio for splitting positive examples for every query to create our training and validation. The data statistics are provided below. We will discuss about the triplets in the upcoming sections.

| Dataset | #Images | #Queries | #Training Triplets | #Validation Triplets |
|---------|---------|----------|--------------------|----------------------|
| Oxford  | 5042    | 55       | 3373               | 831                  |
| Paris   | 6412    | 55       | 13230              | 3421                 |

## Problem Description
As mentioned in the introduction, the problem can be formulated as follows. 
Given a dataset D<sub>n</sub> = {
(q<sub>1</sub>, p<sub>11</sub>, ,p<sub>12</sub> ,p<sub>13</sub> , … , p<sub>1m</sub>),
(q<sub>2</sub>, p<sub>21</sub>, ,p<sub>22</sub> ,p<sub>23</sub> , … , p<sub>2k</sub>),
.....,
(q<sub>n</sub>, p<sub>n1</sub>, ,p<sub>n2</sub> ,p<sub>n3</sub> , … , p<sub>nr</sub>),
}

where q<sub>x</sub> indicates the x<sup>th</sup> query image and p<sub>xk</sub> indicates the k<sup>th</sup> positive example for the query q<sub>x</sub>. Do note that the number of positive examples for each query are not the same.

Given this dataset, our goal is to learn an embedding from these images to a compact latent space where cosine similarity between two learned embeddings correspond to a ranking measure for image retrieval task.

## Loss function
We leverage on a siamese architecture that combines three input streams with a triplet loss. We make use of triplet loss because this has shown to be more effective for ranking problems. 

To formally describe, triplet loss is a loss function where a baseline (anchor, in our case the query image) is compared to a positive (as per annotation) image and a negative image. The triplet loss minimizes the distance from the anchor image to the positive image and maximizes the distance from the anchor image to the negative image over all the triplets in the dataset.  It is formally described below.

![alt text](https://github.com/keshik6/deep-image-retrieval/blob/master/readme_pics/triplet_loss.png)

Where f<sub>ia</sub>, f<sub>ip</sub> and f<sub>in</sub> corresponds to the i<sup>th</sup> anchor, positive and negative embeddings respectively. We use a margin $\alpha$ to separate the embeddings.

![alt_text](https://github.com/keshik6/deep-image-retrieval/blob/master/readme_pics/triplet_learning.png)

Do note that training is quite expensive due to the fact that optimization is directly performed on the triplet space, where the number of possible triplets for training is cubic in the number of training examples.

## How to choose triplets?
A major problem with training triplet optimization problems lies in how the triplets are being chosen. For this specific problem, since we are not given any negative examples for the query, many attempts tend to choose negative examples (that excludes anchor and positive examples) randomly from the dataset and form triplets to be trained on. While this is a reasonable method, we need to show semi-hard examples to the algorithm so that it learns some quantifiable information through parametrization.
Consider the negative examples randomly sampled for the following anchor image.

![alt_text](https://github.com/keshik6/deep-image-retrieval/blob/master/readme_pics/all_souls_000051.jpg)

![alt_text](https://github.com/keshik6/deep-image-retrieval/blob/master/readme_pics/neg_ex1.jpg)

As you can clearly see, a large portion of images chosen to be negative examples are too easy, meaning that the algorithm doesn’t need to make any effort to learn to discriminate between the positive and negative examples.

We thought about both the data and the problem of choosing triplets quite carefully and decided to choose the negative images that have the highest structural similarity against the anchor as our negative examples when creating triplets.

**What is structural similarity?**  
Structural similarity measures the perceptual difference between two images. It considers image degradation as perceived change in structural information. The SSIM formula is a weighted sum based three comparison measurements between the 2 images, namely, luminance, contrast and structure. See appendix section for references.

## Methodology
Given an anchor image, we consider 500x500 center crop of anchor image against all the other non-positive images in the dataset, center cropped to 500x500 and measure the structural similarity. We select the top 500 images with the largest structural similarity as our negative example pool. 

Given this methodology, consider the hard-negatives chosen for the same query image.

![alt_text](https://github.com/keshik6/deep-image-retrieval/blob/master/readme_pics/neg_ssim_ex1.jpg)

As you can see, these examples are hard-negative examples that can allow our algorithm to learn better embeddings. In terms of implementation, we processed the query images to select top 500 negative images based on structural similarity offline and these are annotated as ‘bad’ files.

## Deep Learning Architecture
Deep neural networks have proven to be good feature extractors in the recent time since they carry out representation learning as well without any hand-engineered features. Hence, we decided to use a Resnet50 backbone as our feature extractor network where we removed the Global Average pooling layer and the fully connected layer. An example of the architecture is shown below.
