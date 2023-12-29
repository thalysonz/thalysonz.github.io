---
title: "Tips for Analyzing Obfuscated Code"
date: 2023-12-28
# weight: 1
# aliases: ["/first"]
tags: ["code","en"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Tips for Analyzing the PixPirate Malware Code"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://cdn-images-1.medium.com/max/800/1*GWZv7nO5EqwGKzWNLJCJ8Q.jpeg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/thalysonz.github.io/posts/tips-for-analyzing-obfuscated-code"
    Text: "s" # edit text
    appendFilePath: true # to append file path to Edit link
---


## Introduction

I have completed my static analysis of the PixPirate malware code. A significant part of this period was dedicated to deobfuscating the JavaScript files used by the malware, which made considerable efforts to complicate analysis by third parties.

As can be seen in the images below, all files in "project.zip", executed by the malware using AutoJS (a tool responsible for handling JS automation), are obfuscated. Therefore, I decided to take advantage of my current focus on this topic and provide some tips that might be helpful if you encounter any obfuscated JavaScript code in the future.


![IMG1](https://cdn-images-1.medium.com/max/800/0*H1U5znQyzgslZECB.png)

Code obfuscation is the process of making it less readable to humans. It complicates the understanding of the code's functionalities, making it more difficult to copy or modify. This is achieved through the application of obfuscation techniques, which include renaming variables, modifying the code structure, inserting unnecessary or irrelevant code, and removing comments.

Our job here is to make the above code readable again, so we can understand how the malware operates. So, let's get started.


### **1. Formatting the Code**

In the images above, the three files are essentially unreadable. However, formatting or restructuring the code would greatly aid our process, as it allows us to identify where a function starts or ends, separate functions, and thus, facilitate the next steps.

Below is an example of one of the files before and after formatting.



![IMG2](https://cdn-images-1.medium.com/max/800/0*eSw5f1W4ejdxkLib.png)
![IMG3](https://cdn-images-1.medium.com/max/800/0*VbPWBj-w8vcsHB95.png)

### **2. Renaming**

The next step will be to change the names of functions, classes, variables, interfaces (or whatever we encounter) from something strange to something we can easily identify.

In the following example, we showcase some variables, constants, and functions, among other elements, with names in hexadecimal or random values, aimed at complicating the understanding of the code.


![IMG4](https://cdn-images-1.medium.com/max/800/1*r3KG0kugsPm4OZyjCcQ3eA.png)

Here we have a function that essentially returns an array of strings. We'll start by renaming the function from a1_0x13b5 to mw_FunctionReturnStrings. In VS CODE, by right-clicking and selecting “Change All References”, we can modify all references to this function.

![IMG5](https://cdn-images-1.medium.com/max/800/1*PPkedJg0CioISgRWsfWORA.png)


We can continue renaming the code according to its functionality, and we'll end up with something like the images below.

![IMG6](https://cdn-images-1.medium.com/max/800/1*LW_Ol5LGvj5QS0bGZooZ2A.png)

![IMG6](https://cdn-images-1.medium.com/max/800/0*qkTWmDifL23lvSuI.png)

![IMG6](https://cdn-images-1.medium.com/max/800/1*vn9Abi_ECIKsHzwah_tBYA.png)

![IMG6](https://cdn-images-1.medium.com/max/800/0*ozB6hWO2b4SMg_qK.png)


### **3. Understanding the Code**

Now that we have a readable code, we just need to understand what all this code is doing. To do this, we can follow the function calls throughout the application. In the image below, I used [playcode.io](https://playcode.io) to visualize the result of this array call, which is an important element of this code.

![IMG6](https://cdn-images-1.medium.com/max/800/1*tehhaJXOk_vtyXF17CnHvg.png)


### **4. Application Flow**


This is a time-consuming process in which you will put together everything that's been written here and try to understand the flow as a whole, noting that one file may call another file and so on. In some cases, we will encounter code that is just junk, after all, we are dealing with obfuscation. But with the tips above, understanding the next obfuscated files you come across will be easier.

In the image below, I demonstrate the end of our JS code, where basically all this code was used to call the file "permission/empower.js". Thus, I show that we moved from something unreadable to a much more readable code and managed to understand the flow that the application follows with this code. Now, we just need to do this with the rest of the code.

![IMG6](https://cdn-images-1.medium.com/max/800/0*CsffVrvvv_vRJ9NB.png)