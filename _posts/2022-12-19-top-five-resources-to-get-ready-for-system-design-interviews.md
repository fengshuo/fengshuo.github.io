---
title: Top 5 Resources to Get Ready for System Design Interview
categories:
- Backend
- Systems
tags:
- systemdesign
- interview
author_profile: true
classes: wide
---

System design interview has gained popularity among hiring companies in recent years, and there seems to be a trend that it’s going to be a common module in technical interviews just like data structures and algorithms(even though it is still controversial) .Unlike data structures and algorithms, system design interview is often open-ended, which means the interviewer can probe in many possible directions in less than 45min or 60min, this makes the interview preparation a daunting task.

We could try our best at work to gain more hands-on system design experience to increase our chance of acing the interview, this is a doable solution but it will take quite some time to accumulate  these knowledges or experiences, if we only have a short period of time to prepare for system design interviews, it is a different story. Here are the top 5 resources I’ve found so far:

## 1. **[Designing Data-Intensive Applications](https://amzn.to/3PIYgcT)**

This book has been recommended many times in numerous places, and it is indeed a great start to understand distributed system. Software keeps changing, but the fundamental principles remain the same. The book discusses the pros and cons of various technologies for processing and storing data.

### Who should use it?

If you don’t have much distributed system design knowledge or system design experience, and you have sufficient time to prepare, this book will give a good amount of exposure to what should you consider when designing a data intensive system and the trade offs.

## 2 **[The System Design Primer](https://github.com/donnemartin/system-design-primer)**

![https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dgopberupnuyuu413thk.png](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dgopberupnuyuu413thk.png)

This is probably  the most popular open source system design repo. It covers the fundamental concepts used in system design, a basic framework to approach system design interview questions, as well as applying this framework to solve some common system design questions.

### Who should use it?

If you are going to have an interviewing soon and you don’t have a lot time to learn a lot of things, this primer repo should be handy

## 3 **[System Design Interview – An insider's guide Volume 1](https://amzn.to/3PE39ns), [Volume 2](https://amzn.to/3YA1SlC)**

These books provide step by step detailed guide on how to answer common system design questions in an interview setting. I like how it applies the framework it suggested in every example and the trade offs parts a lot. The first book covers some common system design interview questions, the second book has much  more content and the trade off comparison is deeper

I’ve also used the [Grokking the System Design Interview](https://www.educative.io/courses/grokking-the-system-design-interview) course when there weren’t many other options available to study system design, however looking back now, I think it would be great if it doesn’t spent too much paragraphs on calculating the numbers since in real interviews, you just don’t have that much time to do these calculations and it is supposed to be a back of the envelope estimations.(However educative has released a few new versions of system design courses, I didn't read any these newer versions but hopefully they are better than the initial version).

### Who should use it?

If you are okay with spending a bit money to get the book(s), and your main purpose is to pass the system design interview with flying colours, not to also learn designing distributed system along the way. Then these books are worth the investment.

## 4 System Design Courses

If you’ve ever searched system design videos on YouTube, then most likely you’ve seen this SystemDesignInterview [channel](https://www.youtube.com/@SystemDesignInterview), and if you’ve watched any of the videos in this channel, then most likely you would find it useful and in depth. Good news is that the creator recently release a [System Design for Interviews and Beyond](https://systemdesignthinking.thinkific.com/courses/system-design-for-interviews-and-beyond) course that covers system design interview extensively.

### Who should use it?

If you are a video-watching type of learner, and you’ve already watched the creator’s videos(you liked them), then you don’t wanna miss this course.

## 5 Practice → learn → practice, rinse and repeat

After learning system design knowledge from any of the above resources, the next essential step is to do mock interviews as many as you can, so that you get to practise applying the framework you learned, such as clarifying requirement in an interview setting, how to communicate trade offs with the interviewer etc.

I’ve used two platforms for mock interviews, one is [Pramp](https://www.pramp.com/#/) where get paired randomly with a peer then you interview each other and gather feedback, it’s free(recently acquired by [Exponent](https://blog.tryexponent.com/pramp-joining-exponent/)), although for system design interview, sometimes you get an experienced peer sometimes you are the more experienced peer. If you really want more predictable interviewers, especially when you want to interview for the next level, I would say [interviewing.io](http://interviewing.io/) has interviewers who are more experienced but the session is a bit pricy.

Hope this helps you to start your system design journey!
