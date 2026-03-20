DOCS = \
	00_introduccion/introduccion \
	01_typescript_para_pythonistas/typescript \
	02_postgresql_esencial/postgresql \
	03_dominio_pagos_recurrentes/dominio_toku \
	04_webhooks_e_integraciones/webhooks \
	05_fastapi_avanzado/fastapi \
	06_simulacros_entrevista/simulacros
	07_entrevista_tecnica/entrevista_tecnica

.PHONY: all clean

all:
	@for doc in $(DOCS); do \
		echo "Compilando $$doc..."; \
		cd $$(dirname $$doc) && \
		pdflatex -interaction=nonstopmode $$(basename $$doc).tex > /dev/null 2>&1 && \
		pdflatex -interaction=nonstopmode $$(basename $$doc).tex > /dev/null 2>&1 && \
		echo "  OK: $$(basename $$doc).pdf" && \
		cd - > /dev/null; \
	done
	@echo ""
	@echo "PDFs generados en cada carpeta."

clean:
	@for doc in $(DOCS); do \
		dir=$$(dirname $$doc); \
		base=$$(basename $$doc); \
		rm -f $$dir/$$base.{aux,log,toc,out,fls,fdb_latexmk}; \
	done
	@echo "Archivos auxiliares eliminados."
