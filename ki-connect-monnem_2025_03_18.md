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

Now for the things that I can already write down:













