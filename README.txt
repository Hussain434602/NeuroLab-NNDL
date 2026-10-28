NeuroLab - NNDL project submission package

1_project_code/neurolab/     Full project (library, experiments, web app, tests, Dockerfile). Start with its README.md.
2_report/                    NeuroLab_Report.docx (editable) and a PDF preview. Yellow fields must be filled in.
3_figures_and_results/       Copies of every figure and result file generated so far.
4_viva_and_notes/            Viva questions for both members and the report outline.

STILL TO DO ON YOUR MACHINE (needs internet / PyTorch / FastAPI):
 1. pip install -r 1_project_code/neurolab/requirements.txt
 2. Put ENB2012_data.xlsx into 1_project_code/neurolab/data/ (or let ucimlrepo download it)
 3. python -m experiments.train_energy         -> real Energy results (check heating-load test R2 >= 0.95)
 4. python -m experiments.compare_optimizers   -> convergence table for the real dataset
 5. Fill the yellow fields in the report: names, repo URL, deployment link, Tables 5 and 7, discussion text
 6. pytest, then: uvicorn app.main:app --port 8000
 NOTE: the web app, PyTorch comparison and real-data numbers were NOT run in the sandbox where this was built.
