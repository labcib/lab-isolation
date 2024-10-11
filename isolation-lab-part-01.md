# Introduction

This lab is divided into three parts. In each part, you will experiment with various aspects of software Isolation using Docker, a popular platform for packaging applications and their dependencies into lightweight, portable containers, each in its own isolated environment[^P11]. 

Each of the different parts are intended to be roughly a practical (PL) class, but your mileage may vary; feel free to do it at your own pace, but don’t forget to be thorough on your observations and report. Each part has a setup section indicating what you will need to perform the lab.

Submit a lab report that describes the procedures you followed. Please explain noteworthy or unexpected observations. Don’t forget to address all the questions (in bold) in this lab description. Provide details on how you arrived at the conclusions/results (do not just paste the result). The lab report can be developed in a team of up to 4 members (exceptions must be approved). See the lab report submission dates in Moodle.

## Criteria

The Lab will be graded using the following rubric. Please use this detailed rubric as guidance to create a good, detailed report. If the assignment is deemed unacceptable in a given criterion, a 0 will be given to that respective criterion.

| **Criteria**                   | **weight**      | **Exemplary**  **(100%)**                                    | **Accomplished**  **(75%)**                                  | **Satisfactory**   **(50%)**                                 | **Poor**  **(25%)**                                          |
| ------------------------------ | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Organization and  Language** | 10%             | Good organization, easy to follow, and related to the lab activities.  No major language  (Grammar, Usage, Mechanics, Spelling) errors. | Organized, but points are jumpy or hard to follow.  There are only one or two important language errors. | Some organization; points  jump around; hard to follow.  More than two important  errors. | Poorly organized; no logical progression; hard to connect with lab activities and logical flow.  Numerous errors distract from understanding. |
| **Questions 1-15**             | 80%  (~6% each) | Complete answer  touching all the relevant points, with no or very minor incorrections. | Most relevant points were covered, but there were some small incorrections. | Does not address most key points. There are some major incorrections. | Unable to provide  specific details. Mostly incorrect details/facts. |
| **Question 16**                | 10%  (extra)    | Supporting details  specific to the exploit; Support information and explanations are  consistently accurate.  Demonstration fully  working. | Some details are  non-supporting or marginal to the exploit, or some facts are inaccurate or  inconsistent. Demonstration fully working or with minor issues. | Details and/or facts are somewhat sketchy; Do not support exploit, or very marginally.  No working demonstration or significant issues. | Unable to provide  specific details. No demonstration or significant issues with it. |
| **Question 17**                | 10%             | Complete list of good practices covering the most relevant aspects (e.g., image security, containers, host,  service). Support information and explanations are consistently accurate. | Mostly complete; No more than a couple of points missing or incomplete. Some supporting information  missing | Some relevant points  missing and/or a lot of supporting information missing. | Unable to provide specific details.                          |

# Part 1: Keeping Secrets OUT of Docker Images

Researchers[^P12] have found that many publicly available Docker images contain secrets, such as private keys, API keys and more. Let’s start by spending about 10-15 minutes having a look at the research paper. You only need to get a brief overview of it.

## Setup

For this part of the Lab, you will need a computer with Docker installed. Other tools you might need (like a way to unpack tar files), should be available in most OSes.

## Start: Noob Mistake

Someone at Insecure LabsTM thought it was a good idea to ask Scott, the intern, to create the container for their latest product (a javascript application running on Node.js). He followed the [Docker getting started guide](https://docs.docker.com/get-started/workshop/02_our_app/) and adopted a similar Dockerfile:

```
FROM  node:18-alpine
WORKDIR /app  
COPY . .  
RUN yarn install  --production  
CMD  ["node", "src/index.js"]  EXPOSE 3000  
```

The application uses some credentials at runtime and creates an encrypted secure storage for them. However, during “yarn install”, it needs to have the following .env file (don’t question these design decisions, Tom’s a [super-genious](https://thedailywtf.com/articles/the-inner-json-effect?ref=gorillasun.de)!): 

```
API_KEY=<some  secret api key>  
USER=<some  secret user>  
```

This is how the folder of the application looks like when Scott builds the container:

```
├── insecure-labs/
│ ├── .env
│ ├── package.json
│ ├── README.md
│ ├── src/
│ └── yarn.lock
```

The image is publicly available in Docker Hub:
- https://hub.docker.com/repository/docker/isepdei/insecurelabs01/general

**Question** **1.** **What is the issue here?**

**Question** **2.** **Can you recover the actual secrets from the image? Detail how.**

## Fixed it!

Tom, the senior engineer, quickly spotted Scott’s mistake: “Let me fix that for you!”. Here is the Dockerfile Tom wrote:

```
FROM  node:18-alpine  
WORKDIR /app  
COPY . .  
RUN yarn install  --production  
RUN rm .env # .env  only needed during yarn install
CMD  ["node", "src/index.js"]  EXPOSE 3000  
```

Scott created a new API key and invalidated the old one. The resulting image is publicly available in Docker Hub:
- https://hub.docker.com/repository/docker/isepdei/insecurelabs02/general

**Question** **3.** **What do you think of Tom’s fix?**

**Question** **4.** **Is it possible to recover the new API key now[^P13] ? Provide detailed instructions.**


**Question** **5.** **So, how would you go about doing Scott’s job? Would you do something different? Explain in detail.**

------

[^P11]: https://docs.docker.com/engine/security/

[^P12]: Markus Dahlmanns, Constantin Sander, Robin Decker, and Klaus Wehrle. 2023. Secrets Revealed in Container Images: An Internet-wide Study on Occurrence and Impact. In Proceedings of the 2023 ACM Asia Conference on Computer and Communications Security (ASIA CCS '23). Association for Computing Machinery, New York, NY, USA, 797–811. https://doi.org/10.1145/3579856.3590329. **Available in Moodle**.

[^P13]: Many sources on the Web to help you, but beware that very few are completely accurate; you will have to get it right by combining multiple sources. Here is a hint: you might want to take a **Dive**: https://github.com/wagoodman/dive **and** save the Docker image to a .tar file: https://docs.docker.com/reference/cli/docker/image/save/


