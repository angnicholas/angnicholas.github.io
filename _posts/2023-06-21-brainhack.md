---
title: Brainhack 2023
date: 2023-06-21 19:26:00 -0
categories: [projects]
tags: [hackathons, tech, machine-learning]
---

In May-June 2023, I participated in [Brainhack](https://www.dsta.gov.sg/brainhack), a Machine Learning competition organised by DSTA, which involved completing several Computer Vision and Audio Processing tasks in a team of four. During the competition, I was in charge of developing Machine Learning models for Audio Processing tasks. Our team achieved fourth place in the finals and attained the Most Innovative award.

## Qualifiers: Automatic Speech Recognition

During the Qualifying round, I developed an Automatic Speech Recognition system that could transcribe audio inputs of recorded English speech in a Singaporean accent. There was a set of 3750 annotated samples of English speech in Singaporean accent and transcriptions had to be made on an unlabelled set of 12000. Performance would be judged by Word Error Rate (WER). All model training had to be executable on a free online hardware platform such as Google Colab.

### Approach and Brainstorming

The first step to tackling the challenge was to explore the data and the tools available in order to brainstorm for ideas. To obtain a baseline metric, I ran the unlabelled data through an off-the-shelf Wav2Vec2 model with pretrained weights. Wav2Vec2 is a model which generates output tokens based on characters rather than words. As a result, observation of the model outputs revealed that there were many spelling errors that could be resolved simply with a dictionary approach. Mapping the outputs using an automatic spellchecker seemed like a sensible idea, which further brought the WER down. The combination of these preliminary approaches established a baseline WER of 0.12 which further models need to outperform.

Additionally, I explored the samples in the dataset. Most samples were short clips of around 5 seconds long that typically contained a short phrase or sentence. The presence of sentence structure in each sample suggested the possibility of taking advantage of Language Models (LMs) or grammatical correction tools to further improve performance. The samples also seemed to be of fairly high quality and were similar to one another. This suggested the potential of using data augmentation techniques on the train set to widen the coverage of the input domain, improving the model’s ability to generalise. 

### Development

With this in mind, I began the process of fine tuning by performing a randomised train-test (80-20) split on the data. Throughout training, I monitored the validation loss, training loss and validation accuracy to choose the appropriate number of epochs to prevent the model from overfitting or underfitting. I also adjusted the learning rate and amount of augmentation to a level that produced the best performance. 

After these experiments and adding more post-processing steps to account for grammatical errors in sentences, the final model achieved a a WER of 0.004, a 30-fold improvement from the initial model.

### Challenges

A principal challenge was the limitation of hardware resources. Managing the entire codebase was more challenging due to the online notebook environment. The limitation on processor resources meant that caching intermediate results to disk became an important step so that runtime timeouts would not erase progress that has been already performed. 

Another practical challenge was that a full run of training takes a long time (more than 2 hours), resulting in a high time cost associated with any logical bugs in the code. Combined with a restriction on the number of submissions I could make to the evaluation server in a given day, I had to be quite deliberate in the experimental set-up.

One strategy I employed to reduce the risk of failure was to construct small subsections (of around 50 samples) of the training, validation and testing data. These small subsections would be passed through the model each time I made a major change to the codebase so I could quickly test that the changes I made did not break functionality. The number 50 was chosen to be just above the batch size (32), to ensure near-100% code coverage. After this preliminary check is done, then I will pass the full dataset through.

I also quickly discovered that it is essential to only change one aspect of the pipeline in between model runs, in order to ascertain that any change in performance (the dependent variable) is solely due to that single change (the independent variable). Running many of these small changes and testing would then allow me to piece together a clearer idea of what combination of changes would affect the overall result. This is the scientific aspect of Machine Learning.

I also recognised the importance of tracking metrics and testing my assumptions in order to have a clear picture of what was going on. For example, an initial assumption I had was that the labelled data and unlabelled data were drawn from the same distribution. However, a large disparity between the validation accuracy and the testing accuracy seemed to suggest that might not be true. It is important to recognise these assumptions and challenge them under testing to prevent them from affecting the model.

## Finals

![Robot navigating maze](/assets/images/2023-06-21-brainhack/capture.jpg)
*Robot performing Computer Vision Task while navigating maze*

During the finals, we were tasked with integrating our AI models with a robotic system that would navigate a maze while performing the AI tasks at a series of checkpoints, gaining points along the way. In addition to making our AI systems run on the robot, two new AI tasks were introduced which were digit recognition and speaker identification.

### System Robustness

In addition to training new models, it was important to manage the robustness of the system due to it being a real-time system. As such, it was critical to think about the various ways in which the system could fail in a real-time scenario, and how it could possibly recover from error. For example, the AI system could fail for some unexpected reason, or take too long to execute - in which case a mechanism of halting execution and recovering from failure was necessary. In order to address this issue, I placed try-except clauses around various parts of the code in order to catch any errors and specify a fallback result (such as returning a random answer) to allow the robot to skip the task and move on to the next one. Additionally, I implemented a timing mechanism to stop any inference that exceeds a predetermined time frame as our robot was given a fixed amount of time to navigate the maze. Other fallback strategies were also put inplace to allow the robot to backtrack in the event it hit an obstacle or otherwise got stuck while navigating the maze, to hopefully reach a location where it is able to try navigating again. The robot was able to use this mechanism to succesfully backtrack to a known location when it got stuck, demonstrating the necessity for analysing various failure modes when designing a real-time system as opposed to problem-style coding which I was previously used to (eg. solving Hackerrank problems) where programmer intervention due to a bug is always possible.  

### Speaker Identification: Few-shot learning
For the speaker identification task, we were presented a new type of challenge which involved accurately identifying the identity of a speaker from a short audio clip of around 15 seconds from a group of eight individuals, given a training data set of one audio clip for each individual in the group. Due to the lack of available training data, this posed a few-shot learning problem. Initially, I experimented with several different methods - such as splitting the audio clips up into several segments and feeding their spectrograms into a classifier. However, this method failed to generalise due to the lack of data - as the models were likely overfitting on noise. The problem seemed to suggest looking towards similarity-based models instead which involved embedding the audio clip as a vector inside some kind of space where the cosine similarity between two vectors corresponds to how similar the audio clips are with regards to speaker identity. Using such methods, I was able to achieve a 20% accuracy on the local validation set as well as in real-time performance during the finals competition, which was not much better than chance. However, I still felt the problem was very interesting, and something I would wish to investigate further in the future.

## Moving Forward

Depsite not placing in the top three teams, I am very grateful for this experience as I have learnt many new techniques in Machine Learning - both within the Audio Processing domain, as well as from my teammates who used Siamese Neural Networks to perform Object Re-identification in the Computer Vision Task. The inclusion of the robotic navigation system was also a fun twist that added an additional dimension of complexity, and I believe it gave me a glimpse into the process of developing critical systems (such as flight control systems, etc.) where systems are expected to function correctly without programmer intervention and the cost of failure is high.

I look forward to expanding my knowledge in Machine Learning and look forward to taking part in more competitions like this in the future.