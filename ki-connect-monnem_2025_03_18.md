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

- The first figure from the paper shows a good overview

  - ![The structure of the RAG system with retrieval and generation components and corresponding four phrases: indexing, search, prompting and inferencing.](https://arxiv.org/html/2405.07437v2/extracted/5707186/rag-structure.png "Title")












