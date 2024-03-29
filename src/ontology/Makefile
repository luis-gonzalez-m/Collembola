# ----------------------------------------
# Makefile for clao
# Generated using ontology-development-kit
# ODK Version: v1.2.29
# ----------------------------------------
# IMPORTANT: DO NOT EDIT THIS FILE. To override default make goals, use clao.Makefile instead


# ----------------------------------------
# More information: https://github.com/INCATools/ontology-development-kit/


# ----------------------------------------
# Standard Constants
# ----------------------------------------
# these can be overwritten on the command line

URIBASE=                    http://purl.obolibrary.org/obo
ONT=                        clao
ONTBASE=                    $(URIBASE)/$(ONT)
EDIT_FORMAT=                owl
SRC =                       $(ONT)-edit.$(EDIT_FORMAT)
CATALOG=                    catalog-v001.xml
ROBOT=                      robot --catalog $(CATALOG)
RELEASEDIR=                 ../..
REPORTDIR=                  reports
TEMPLATEDIR=                ../templates
TMPDIR=                     tmp
SCRIPTSDIR=                 ../scripts
SPARQLDIR =                 ../sparql
COMPONENTSDIR =             components
REPORT_FAIL_ON =            None
REPORT_LABEL =              -l true
REPORT_PROFILE_OPTS =       
OBO_FORMAT_OPTIONS =        
SPARQL_VALIDATION_CHECKS =   equivalent-classes owldef-self-reference
SPARQL_EXPORTS =             basic-report class-count-by-prefix edges xrefs obsoletes synonyms
ODK_VERSION_MAKEFILE =      v1.2.29

TODAY ?=                    $(shell date +%Y-%m-%d)
OBODATE ?=                  $(shell date +'%d:%m:%Y %H:%M')
VERSION=                    $(TODAY)
ANNOTATE_ONTOLOGY_VERSION = annotate -V $(ONTBASE)/releases/$(VERSION)/$@ --annotation owl:versionInfo $(VERSION)
OTHER_SRC =                 $(PATTERNDIR)/definitions.owl 
ONTOLOGYTERMS =             $(TMPDIR)/ontologyterms.txt
PATTERNDIR=                 ../patterns
DOSDP_SCHEMA=                http:// # change to PURL when ready.
PATTERN_TESTER=              simple_pattern_tester.py
DOSDPT=                      dosdp-tools
PATTERN_RELEASE_FILES=       $(PATTERNDIR)/definitions.owl $(PATTERNDIR)/pattern.owl



FORMATS = $(sort  owl obo json owl)
FORMATS_INCL_TSV = $(sort $(FORMATS) tsv)
RELEASE_ARTEFACTS = $(sort $(ONT)-base $(ONT)-full $(ONT)-simple $(ONT)-basic $(ONT)-base $(ONT)-full)

# ----------------------------------------
# Top-level targets
# ----------------------------------------

.PHONY: .FORCE

.PHONY: all
all: odkversion all_imports patterns all_main all_subsets sparql_test all_reports all_assets

.PHONY: test
test: odkversion sparql_test all_reports 
	$(ROBOT) reason --input $(SRC) --reasoner ELK  --equivalent-classes-allowed all --exclude-tautologies structural --output test.owl && rm test.owl && echo "Success"

.PHONY: odkversion
odkversion:
	echo "ODK Makefile version: $(ODK_VERSION_MAKEFILE) (this is the version of the ODK with which this Makefile was generated, not the version of the ODK you are running)" &&\
	echo "ROBOT version (ODK): " && $(ROBOT) --version

$(TMPDIR) $(REPORTDIR) :
	mkdir -p $@

## -- main targets --
##
## By default this is the cross-product of {ont, ont-base} x FORMATS

MAIN_PRODUCTS = $(sort $(foreach r,$(RELEASE_ARTEFACTS), $(r)) $(ONT))
MAIN_GZIPPED = 
MAIN_FILES = $(foreach n,$(MAIN_PRODUCTS), $(foreach f,$(FORMATS), $(n).$(f))) $(MAIN_GZIPPED)

.PHONY: all_main
all_main: $(MAIN_FILES)

## -- import targets --
##
## By default this is the cross-product of IMPORT_MODULES x FORMATS


IMPORTS = 

IMPORT_ROOTS = $(patsubst %, imports/%_import, $(IMPORTS))
IMPORT_OWL_FILES = $(foreach n,$(IMPORT_ROOTS), $(n).owl)
IMPORT_FILES = $(IMPORT_OWL_FILES)


.PHONY: all_imports
all_imports: $(IMPORT_FILES)

## -- subset targets --
##
## By default this is the cross-product of SUBSETS x FORMATS
## Note we also include TSV as a format


SUBSETS = 

SUBSET_ROOTS = $(patsubst %, subsets/%, $(SUBSETS))
SUBSET_FILES = $(foreach n,$(SUBSET_ROOTS), $(foreach f,$(FORMATS_INCL_TSV), $(n).$(f)))

.PHONY: all_subsets
all_subsets: $(SUBSET_FILES)

OBO_REPORT =  $(SRC)-obo-report
REPORTS = $(OBO_REPORT)
REPORT_FILES = $(patsubst %, $(REPORTDIR)/%.tsv, $(REPORTS))

.PHONY: robot_reports
robot_reports: $(REPORT_FILES)

.PHONY: all_reports
all_reports: all_reports_onestep $(REPORT_FILES)

$(REPORTDIR)/validate_profile_owl2dl_%.txt: % | $(REPORTDIR)
	$(ROBOT) validate-profile --profile DL -i $< -o $@
.PRECIOUS: $(REPORTDIR)/validate_profile_owl2dl_%.txt

.PHONY: validate_profile_%
validate_profile_%: $(REPORTDIR)/validate_profile_owl2dl_%.txt

## -- all files/assets --

ASSETS = \
  $(IMPORT_FILES) \
  $(MAIN_FILES) \
  $(REPORT_FILES) \
  $(SUBSET_FILES)

RELEASE_ASSETS = \
  $(MAIN_FILES) \
  $(SUBSET_FILES)

.PHONY: all_assets
all_assets: $(ASSETS)


.PHONY: show_assets
show_assets:
	echo $(ASSETS)
	du -sh $(ASSETS)


# ----------------------------------------
# Release Management
# ----------------------------------------

KEEPRELATIONS=keeprelations.txt
CLEANFILES=$(MAIN_FILES) $(SRCMERGED)
# This should be executed by the release manager whenever time comes to make a release.
# It will ensure that all assets/files are fresh, and will copy to release folder

.PHONY: prepare_release
prepare_release: $(ASSETS) $(PATTERN_RELEASE_FILES)
	rsync -R $(RELEASE_ASSETS) $(RELEASEDIR) &&\
  mkdir -p $(RELEASEDIR)/patterns &&\
  cp $(PATTERN_RELEASE_FILES) $(RELEASEDIR)/patterns &&\
  rm -f $(CLEANFILES) &&\
  echo "Release files are now in $(RELEASEDIR) - now you should commit, push and make a release on your git hosting site such as GitHub or GitLab"

.PHONY: prepare_initial_release
prepare_initial_release: prepare_release
	cd $(RELEASEDIR) && git add $(RELEASE_ASSETS)

# ----------------------------------------
# Import modules
# ----------------------------------------
# Most ontologies are modularly constructed using portions of other ontologies
# These live in the imports/ folder

# ------------------------
# Imports: Seeding system 
# ------------------------

# seed.txt contains all referenced entities
IMPORTSEED=$(TMPDIR)/seed.txt
SRCMERGED=$(TMPDIR)/merged-$(SRC)
PRESEED=$(TMPDIR)/pre_seed.txt

$(SRCMERGED): $(SRC)
	$(ROBOT) remove --input $< --select imports --trim false \
		merge  $(patsubst %, -i %, $(OTHER_SRC)) -o $@

$(PRESEED): $(SRCMERGED)
	$(ROBOT) query -f csv -i $< --query ../sparql/terms.sparql $@.tmp &&\
	cat $@.tmp | sort | uniq >  $@

SIMPLESEED=$(TMPDIR)/simple_seed.txt

$(SIMPLESEED): $(SRCMERGED) $(ONTOLOGYTERMS)
	$(ROBOT) query -f csv -i $< --query ../sparql/simple-seed.sparql $@.tmp &&\
	cat $@.tmp $(ONTOLOGYTERMS) | sort | uniq >  $@ &&\
	echo "http://www.geneontology.org/formats/oboInOwl#SubsetProperty" >> $@ &&\
	echo "http://www.geneontology.org/formats/oboInOwl#SynonymTypeProperty" >> $@


ALLSEED = $(PRESEED)  $(PATTERNDIR)/all_pattern_terms.txt \


$(IMPORTSEED):  prepare_patterns  $(ALLSEED) 
	if [ $(IMP) = true ]; then cat $(ALLSEED) | sort | uniq > $@; fi


ANNOTATION_PROPERTIES=rdfs:label IAO:0000115 

# -- Generate Import Modules --
#
# This pattern uses ROBOT to generate an import module
# Generate terms.txt for each import.  (Assume OBO-style Possibly hacky step?)
# Should be able to drop this if robot can just take a big messy list of terms as input.
imports/%_terms_combined.txt: $(IMPORTSEED) imports/%_terms.txt
	if [ $(IMP) = true ]; then cat $^ | grep -v ^# | sort | uniq >  $@; fi


imports/%_import.owl: mirror/%.owl imports/%_terms_combined.txt
	if [ $(IMP) = true ]; then $(ROBOT) query -i $< --update ../sparql/preprocess-module.ru \
		extract -T imports/$*_terms_combined.txt --force true --copy-ontology-annotations true --individuals include --method BOT \
		query --update ../sparql/inject-subset-declaration.ru --update ../sparql/postprocess-module.ru \
		annotate --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) --output $@.tmp.owl && mv $@.tmp.owl $@; fi

.PRECIOUS: imports/%_import.owl

.PHONY: refresh-imports
refresh-imports:
	make IMP=true MIR=true PAT=false IMP_LARGE=true all_imports -B

.PHONY: refresh-imports-excluding-large
refresh-imports-excluding-large:
	make IMP=true MIR=true PAT=false IMP_LARGE=false all_imports -B

.PHONY: refresh-%
refresh-%:
	make IMP=true IMP_LARGE=true MIR=true PAT=false imports/$*_import.owl -B



# ----------------------------------------
# Mirroring upstream ontologies
# ----------------------------------------
#

IMP=true # Global parameter to bypass import generation
MIR=true # Global parameter to bypass mirror generation
IMP_LARGE=true # Global parameter to bypass handling of large imports
PAT=true # Global parameter to bypass pattern generation


# ----------------------------------------
# Subsets
# ----------------------------------------
subsets/%.tsv: subsets/%.owl
	$(ROBOT) query -f tsv -i $< -s ../sparql/labels.sparql $@
.PRECIOUS: subsets/%.tsv

subsets/%.owl: $(ONT).owl
	owltools --use-catalog $< --extract-ontology-subset --fill-gaps --subset $* -o $@.tmp.owl && mv $@.tmp.owl $@ &&\
	$(ROBOT) annotate --input $@ --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) -o $@.tmp.owl && mv $@.tmp.owl $@
.PRECIOUS: subsets/%.owl


subsets/%.obo: subsets/%.owl
	$(ROBOT) convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o $@.tmp.obo && grep -v ^owl-axioms $@.tmp.obo > $@ && rm $@.tmp.obo

subsets/%.json: subsets/%.owl
	$(ROBOT) convert --input $< --check false -f json -o $@.tmp.json &&\
	jq -S 'walk(if type == "array" then sort else . end)' $@.tmp.json > $@ && rm $@.tmp.json


# ----------------------------------------
# Release
# ----------------------------------------
# copy from staging area (this directory) to top-level
.PHONY: release
release: $(ONT).owl $(ONT).obo
	cp $^ $(RELEASEDIR) && cp imports/* $(RELEASEDIR)/imports

# ----------------------------------------
# Sparql queries: Q/C
# ----------------------------------------

# these live in the ../sparql directory, and have suffix -violation.sparql
# adding the name here will make the violation check live.
# NOTE: these will soon be phased out and replaced by robot-report

#  run all violation checks
SPARQL_VALIDATION_QUERIES = $(foreach V,$(SPARQL_VALIDATION_CHECKS),$(SPARQLDIR)/$(V)-violation.sparql)
sparql_test: $(SRC) catalog-v001.xml | $(REPORTDIR)
ifneq ($(SPARQL_VALIDATION_QUERIES),)
	$(ROBOT) verify  --catalog catalog-v001.xml -i $< --queries $(SPARQL_VALIDATION_QUERIES) -O $(REPORTDIR)
endif

# ----------------------------------------
# ROBOT report
# ----------------------------------------
$(REPORTDIR)/%-obo-report.tsv: % | $(REPORTDIR)
	$(ROBOT) report -i $< $(REPORT_LABEL) $(REPORT_PROFILE_OPTS) --fail-on $(REPORT_FAIL_ON) --print 5 -o $@

# ----------------------------------------
# Sparql queries: Exports
# ----------------------------------------

SPARQL_EXPORTS_ARGS = $(foreach V,$(SPARQL_EXPORTS),-s $(SPARQLDIR)/$(V).sparql $(REPORTDIR)/$(V).tsv)
# This combines all into one single command

.PHONY: all_reports_onestep
all_reports_onestep: $(SRC)
ifneq ($(SPARQL_EXPORTS_ARGS),)
	$(ROBOT) query -f tsv -i $< $(SPARQL_EXPORTS_ARGS)
endif
# ----------------------------------------
# Patterns (experimental)
# ----------------------------------------

# Test patterns for schema compliance:

.PHONY: patterns
patterns: all_imports $(PATTERNDIR)/pattern.owl $(PATTERNDIR)/definitions.owl

.PHONY: pattern_clean
pattern_clean:
	echo "Not implemented"

.PHONY: pattern_schema_checks
pattern_schema_checks: update_patterns
	if [ $(PAT) = true ]; then $(PATTERN_TESTER) $(PATTERNDIR)/dosdp-patterns/; fi

#This command is a workaround for the absence of -N and -i in wget of alpine (the one ODK depend on now). It downloads all patterns specified in external.txt
.PHONY: update_patterns
update_patterns: .FORCE
	if [ $(PAT) = true ]; then rm -f $(PATTERNDIR)/dosdp-patterns/*.yaml.1 || true; fi
	if [ $(PAT) = true ] && [ -s $(PATTERNDIR)/dosdp-patterns/external.txt ]; then wget -i $(PATTERNDIR)/dosdp-patterns/external.txt --backups=1 -P $(PATTERNDIR)/dosdp-patterns; fi
	if [ $(PAT) = true ]; then rm -f $(PATTERNDIR)/dosdp-patterns/*.yaml.1 || true; fi


$(PATTERNDIR)/pattern.owl: pattern_schema_checks update_patterns
	if [ $(PAT) = true ]; then $(DOSDPT) prototype --obo-prefixes true --template=$(PATTERNDIR)/dosdp-patterns --outfile=$@; fi

individual_patterns_default := $(patsubst %.tsv, $(PATTERNDIR)/data/default/%.ofn, $(notdir $(wildcard $(PATTERNDIR)/data/default/*.tsv)))
pattern_term_lists_default := $(patsubst %.tsv, $(PATTERNDIR)/data/default/%.txt, $(notdir $(wildcard $(PATTERNDIR)/data/default/*.tsv)))





# Generating the individual pattern modules and merging them into definitions.owl
$(PATTERNDIR)/definitions.owl: prepare_patterns update_patterns dosdp_patterns_default  
	if [ $(PAT) = true ] && [ "${individual_patterns_names_default}" ]   && [ $(PAT) = true ]; then $(ROBOT) merge $(addprefix -i , $(individual_patterns_default))   annotate --ontology-iri $(ONTBASE)/patterns/definitions.owl  --version-iri $(ONTBASE)/releases/$(TODAY)/patterns/definitions.owl --annotation owl:versionInfo $(VERSION) -o definitions.ofn && mv definitions.ofn $@; fi

individual_patterns_names_default := $(strip $(patsubst %.tsv,%, $(notdir $(wildcard $(PATTERNDIR)/data/default/*.tsv))))
dosdp_patterns_default: $(SRC) all_imports .FORCE
	if [ $(PAT) = true ] && [ "${individual_patterns_names_default}" ]; then $(DOSDPT) generate --catalog=catalog-v001.xml --infile=$(PATTERNDIR)/data/default/ --template=$(PATTERNDIR)/dosdp-patterns --batch-patterns="$(individual_patterns_names_default)" --ontology=$< --obo-prefixes=true --outfile=$(PATTERNDIR)/data/default; fi





# Generating the seed file from all the TSVs. If Pattern generation is deactivated, we still extract a seed from definitions.owl
$(PATTERNDIR)/all_pattern_terms.txt: $(pattern_term_lists_default)   $(PATTERNDIR)/pattern_owl_seed.txt
	if [ $(PAT) = true ]; then cat $^ | sort | uniq > $@; else $(ROBOT) query --use-graphs true -f csv -i ../patterns/definitions.owl --query ../sparql/terms.sparql $@; fi

$(PATTERNDIR)/pattern_owl_seed.txt: $(PATTERNDIR)/pattern.owl
	if [ $(PAT) = true ]; then $(ROBOT) query --use-graphs true -f csv -i $< --query ../sparql/terms.sparql $@; fi

$(PATTERNDIR)/data/default/%.txt: $(PATTERNDIR)/dosdp-patterns/%.yaml $(PATTERNDIR)/data/default/%.tsv .FORCE
	if [ $(PAT) = true ]; then $(DOSDPT) terms --infile=$(word 2, $^) --template=$< --obo-prefixes=true --outfile=$@; fi

.PHONY: prepare_patterns
prepare_patterns:
	if [ $(PAT) = true ]; then touch $(PATTERNDIR)/data $(pattern_term_lists_default)  ; fi
	if [ $(PAT) = true ]; then touch $(PATTERNDIR)/data $(individual_patterns_default)  ; fi





# ----------------------------------------
# Release artefacts: export formats
# ----------------------------------------


$(ONT)-base.obo: $(ONT)-base.owl
	$(ROBOT) convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o $@.tmp.obo && grep -v ^owl-axioms $@.tmp.obo > $@ && rm $@.tmp.obo
$(ONT)-base.json: $(ONT)-base.owl
	$(ROBOT) annotate --input $< --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		convert --check false -f json -o $@.tmp.json &&\
	jq -S 'walk(if type == "array" then sort else . end)' $@.tmp.json > $@ && rm $@.tmp.json
$(ONT)-full.obo: $(ONT)-full.owl
	$(ROBOT) convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o $@.tmp.obo && grep -v ^owl-axioms $@.tmp.obo > $@ && rm $@.tmp.obo
$(ONT)-full.json: $(ONT)-full.owl
	$(ROBOT) annotate --input $< --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		convert --check false -f json -o $@.tmp.json &&\
	jq -S 'walk(if type == "array" then sort else . end)' $@.tmp.json > $@ && rm $@.tmp.json
$(ONT)-simple.obo: $(ONT)-simple.owl
	$(ROBOT) convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o $@.tmp.obo && grep -v ^owl-axioms $@.tmp.obo > $@ && rm $@.tmp.obo
$(ONT)-simple.json: $(ONT)-simple.owl
	$(ROBOT) annotate --input $< --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		convert --check false -f json -o $@.tmp.json &&\
	jq -S 'walk(if type == "array" then sort else . end)' $@.tmp.json > $@ && rm $@.tmp.json
$(ONT)-basic.obo: $(ONT)-basic.owl
	$(ROBOT) convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o $@.tmp.obo && grep -v ^owl-axioms $@.tmp.obo > $@ && rm $@.tmp.obo
$(ONT)-basic.json: $(ONT)-basic.owl
	$(ROBOT) annotate --input $< --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		convert --check false -f json -o $@.tmp.json &&\
	jq -S 'walk(if type == "array" then sort else . end)' $@.tmp.json > $@ && rm $@.tmp.json
# We always want a base - even if it is not explicitly configured..
# We always want a full release - even if it is not explicitly configured..
# ----------------------------------------
# Release artefacts: main release artefacts
# ----------------------------------------

$(ONT).owl: $(ONT)-full.owl
	$(ROBOT) annotate --input $< --ontology-iri $(URIBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		convert -o $@.tmp.owl && mv $@.tmp.owl $@

$(ONT).obo: $(ONT).owl
	$(ROBOT) convert --input $< --check false -f obo $(OBO_FORMAT_OPTIONS) -o $@.tmp.obo && grep -v ^owl-axioms $@.tmp.obo > $@ && rm $@.tmp.obo
$(ONT).json: $(ONT)-full.owl
	$(ROBOT) annotate --input $< --ontology-iri $(URIBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		convert --check false -f json -o $@.tmp.json &&\
	jq -S 'walk(if type == "array" then sort else . end)' $@.tmp.json > $@ && rm $@.tmp.json
# -----------------------------------------------------
# Release artefacts: variants (base, full, simple, etc)
# -----------------------------------------------------
SHARED_ROBOT_COMMANDS = 

$(ONTOLOGYTERMS): $(SRC) $(OTHER_SRC)
	touch $(ONTOLOGYTERMS) && \
	$(ROBOT) query --use-graphs true -f csv -i $< --query ../sparql/clao_terms.sparql $@










# base: OTHER sources of interest, such as definitions owl
$(ONT)-base.owl: $(SRC) $(OTHER_SRC)
	$(ROBOT) remove --input $< --select imports --trim false \
		merge $(patsubst %, -i %, $(OTHER_SRC)) \
		 $(SHARED_ROBOT_COMMANDS) annotate --link-annotation http://purl.org/dc/elements/1.1/type http://purl.obolibrary.org/obo/IAO_8000001 \
		--ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) \
		--output $@.tmp.owl && mv $@.tmp.owl $@

# Full: The full artefacts with imports merged, reasoned
$(ONT)-full.owl: $(SRC) $(OTHER_SRC)
	$(ROBOT) merge --input $< \
		reason --reasoner ELK --equivalent-classes-allowed all --exclude-tautologies structural \
		relax \
		reduce -r ELK \
		$(SHARED_ROBOT_COMMANDS) annotate --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) --output $@.tmp.owl && mv $@.tmp.owl $@

# foo-simple: (edit->reason,relax,reduce,drop imports, drop every axiom which contains an entity outside the "namespaces of interest")
# drop every axiom: filter --term-file keep_terms.txt --trim true
#	remove --select imports --trim false \

$(ONT)-simple.owl: $(SRC) $(OTHER_SRC) $(SIMPLESEED)
	$(ROBOT) merge --input $< $(patsubst %, -i %, $(OTHER_SRC)) \
		reason --reasoner ELK --equivalent-classes-allowed all --exclude-tautologies structural \
		relax \
		remove --axioms equivalent \
		relax \
		filter --term-file $(SIMPLESEED) --select "annotations ontology anonymous self" --trim true --signature true \
		reduce -r ELK \
		query --update ../sparql/inject-subset-declaration.ru \
		$(SHARED_ROBOT_COMMANDS) annotate --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) --output $@.tmp.owl && mv $@.tmp.owl $@
# foo-basic: A version of -simple containing only relationships using relations on a configurable whitelist (default = BFO:0000050 (?)). See above (David comment) for explanation.
# removes any axioms that contains one of the ops that not in the whitelist file

$(ONT)-basic.owl: $(SRC) $(OTHER_SRC) $(SIMPLESEED) $(KEEPRELATIONS)
	$(ROBOT) merge --input $< $(patsubst %, -i %, $(OTHER_SRC)) \
		reason --reasoner ELK --equivalent-classes-allowed all --exclude-tautologies structural \
		relax \
		remove --axioms equivalent \
		remove --axioms disjoint \
		remove --term-file $(KEEPRELATIONS) --select complement --select object-properties --trim true \
		relax \
		filter --term-file $(SIMPLESEED) --select "annotations ontology anonymous self" --trim true --signature true \
		reduce -r ELK \
		$(SHARED_ROBOT_COMMANDS) annotate --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) --output $@.tmp.owl && mv $@.tmp.owl $@




.PHONY: validate_idranges
validate_idranges:
	amm $(SCRIPTSDIR)/validate_id_ranges.sc clao-idranges.owl

.PHONY: update_repo
update_repo:
	sh $(SCRIPTSDIR)/update_repo.sh
	


include clao.Makefile