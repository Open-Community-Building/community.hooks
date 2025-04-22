# Cognee Sprint at PyCon PyData 2025

This is a quick summary of what my installation looked like on our server.

Learn about Cognee here:

https://www.cognee.ai/

I was interested in this sprint topic because I like the idea of building a knowledge graph for documents
along the way.
The hope here is that by setting up a stage for a query, a knowledge graph can help selecting
the relevant documents for queries and respond accurately. 

During the sprint I could get it to work on on our GEX44 Hetzner servera for a minimal project.
I want to revisit my notes here to try it out on the Heidelberg city council documents.

Council Analytics:

https://github.com/Open-Community-Building/hackathon-council-analytics/

The sprint was managed by Hande Kafkas from Cognee. 
She quickly got us set up and answered any questions we had, so we could get things done.

So let's dive in to see how far I came:

I logged in to our server with user maik, and and cloned the repository in my home folder.

```shell
ssh maik@altstadt.openheidelberg.de
ssh-add .ssh/gh/id_rsa
git clone git@github.com:topoteretes/cognee.git
mkdir venv
cd venv/
virtualenv .
  creator CPython3Posix(dest=/home/maik/venv, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, via=copy, app_data_dir=/home/maik/.local/share/virtualenv)
    added seed packages: pip==24.0
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
source bin/activate
cd
cd cognee
pip install .
```

