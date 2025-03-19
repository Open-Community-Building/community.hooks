Probieren ist nicht Evaluieren: Bauchgefühl vs. belastbare KI-Bewertungsmethoden

https://www.meetup.com/de-DE/ki-connect-monnem/events/306095687/

"KI connect Monnem" organised an interesting Meetup about modern AI evaluation that I had to visit, and
I also announced it beforehand in our this on our Open Community Building calendar:

https://github.com/Open-Community-Building/community.calendar/edit/main/README.md

There was one talk by Barbara Lampl, a behavioral mathematician from Cologne.

https://www.linkedin.com/in/barbaralampl/

The general observation is that many teams start with something like a RAG LLM as a POC, and are too quickly happy with the results.
That is, until the customer becomes unhappy.

The main takeaway from the talk was that if your RAG LLM is business critical, you better have a plan for systematic evaluations from the get-go:

- LLM-as-a-Judge
- ROUGE and BLEU Score
- nouanced retrieval metrics
- constructing synthetic vs. real data sets 
- what help can platforms like LangSmith oder Langfuse provide?
- integration of user feedback into quantitative evaluation systems? 

We were promised the document used for the talk, and then I can go into more depth about some of these points.

Now for the things that I can already write down so that I do not forget them. I'll get back to this later when I learn more:

- There is one main paper about RAG that you should have read, and the main figure of this paper was used to explain RAG:

  - Evaluation of Retrieval-Augmented Generation: A Survey: https://arxiv.org/abs/2405.07437

- The first figure from the paper shows a good overview (The link will probably break, but you know where to get the paper)

  - ![The structure of the RAG system with retrieval and generation components and corresponding four phrases: indexing, search, prompting and inferencing.](https://arxiv.org/html/2405.07437v2/extracted/5707186/rag-structure.png "Title")

- Another paper that was mentioned was the WritingBench paper, which takes one domain and does the evaluation right:

  - WritingBench: A Comprehensive Benchmark for Generative Writing: https://arxiv.org/abs/2503.05244

- Hugging Face Hallucination Leaderboard

  - https://huggingface.co/spaces/vectara/Hallucination-evaluation-leaderboard

- Seems like it is common knowledge that making a public facing Chat bot can result in drama - Insert your horror story here...

- I learned that experts don't mind hallucinations so much, because they can easily cope with the wrong information

- It seems like the top job profiles that german companies need to hire for, but mostly fail are: Data / AI engineers

- Vibe Coding got some bashing, and I guess I need to learn about it a bit more to join in ;-)

- It sounded reasobable that a lot of improvements for RAG systems can be obtained by addition of good meta data

- An observation was made about many startups seem to focus on question and answer, but there is much more out there, like summarisation and reasoning.

One thing that made me laugh inside was when it was mentioned that it was often the case in the past that researchers did not version their results.
While working in the lab of Roderic Guigó in the CRG in Barcelona, I was the one who made sure that all of the results from all researchers
got immediately added to Git. There is only one researcher who refused to collaborate with me, and guess who had to roll a dice later 
to decide which of the many results they had produced was the right one to be used for to the paper.

- This is the Nature paper I contributed to: An integrated encyclopedia of DNA elements in the human genome: https://www.nature.com/articles/nature11247


