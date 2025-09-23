# Organization of Wizard JSON Files 
note: refer to GenPipes_Wizard.drawio (https://drive.google.com/file/d/1A2Zy3i0v-PikoXVyaDM1rSYpjAbEmRrn/view?usp=sharing) for visualization of the decision trees
---------------------------------
File Structure
---------------------------------

Each wizard file has two top-level keys:

- _meta: Metadata about the file.
- nodes: A list of nodes that make up the decision tree

Example:
{
  "_meta": {
    "entry_point": "start_general_guide",
    "version": 1.0,
    "description": "General guide for GenPipes wizard"
  },
  "nodes": [ ... ]
}

---------------------------------
Node Types
---------------------------------

Every node must have a unique id and a type. The type determines what fields are required.

1. confirm
-----------
Ask the user a yes/no (or multiple-choice) question.

Example:
{
  "id": "start_general_guide",
  "type": "confirm", 
  "question": "Do you need help deploying GenPipes?",
  "options": [
      {
        "label": "Yes",
        "next": {
          "external": "deployment_guide.json",
          "entryPoint": "start_deployment_guide"
        }
      },
      {
        "label": "No",
        "next": "pipeline_help"
      }
    ]
},

Structure:
- question: question to ask the user 
- options: list of choices
  - label: "Yes" or "No"
  - next: node to go to next (either a node ID, or an external JSON reference). 
- for external JSON reference: 
  - external: name of JSON file
  - entrypoint: id of node to enter next 


2. selection
-------------
Lets the user choose from multiple items.

Example:
{
  "id": "pipeline_selection",
  "type": "selection",
  "question": "Select the pipeline you are using:",
  "choices": [
    { "label": "DNA Sequencing Pipeline", "node": "dnaseq_pipeline_selected" },
    { "label": "ChIP Sequencing Pipeline", "node": "chipseq_pipeline_selected" },
    { "label": "CoV Sequencing Pipeline", "node": "covseq_pipeline_selected" },
    { "label": "DNA Sequencing Pipeline", "node": "dnaseq_pipeline_selected" }
  ]
}

Structure:
- choices: 
  - label: option that the user needs to select
  - node: id of node to enter next 

---
Filter/customize list of options that the user picks from (list depends on what variable the user previously selected)

Example:
{
  "id": "protocol_selection",
  "type": "selection",
  "question": "Select the protocol you are using:",
  "choices_cases": [
    {
      "when": { "equals": { "pipeline_name": "chipseq" } },
      "choices": [
        { "label": "ChipSeq (default)", "node": "chipseq_protocol_selected" },
        { "label": "ATACSeq", "node": "atacseq_protocol_selected" }
      ]
    },
    {
      "when": { "equals": { "pipeline_name": "longread_dnaseq" } },
      "choices": [
        { "label": "Nanopore (default)", "node": "nanopore_protocol_selected" },
        { "label": "Revio", "node": "revio_protocol_selected" }
      ]
    }
  ]
}

Structure:
- choices_cases: conditional choices based on variables
  - when ... equals ... [variable, e.g pipeline_name]: depending on what variable (e.g pipeline) the user selects previously, ask the user to select a choice
  - choices: choices that the user needs to select from
    - label: option that the user needs to select
    - node: id of node to enter next

3. set_variable
-----------------
Assigns a variable for later use.

Example:
{
  "id": "chipseq_pipeline_selected",
  "type": "set_variable",
  "variable": "pipeline_name",
  "value": "chipseq",
  "next": "pipeline_choice_message_next_protocol"
}

Structure:
- variable: name of the variable to set
- value: value to assign
- next: id of next node

4. switch
------------
Checks value of a variable. Depending on that variable, jumps to the corresponding node

Example:
{
  "id": "start_protocol_guide",
  "type": "switch",
  "variable": "pipeline_name",
  "cases": {
    "chipseq": {"node": "chipseq_protocol_help"},
    "dnaseq": {"node": "dnaseq_protocol_help"},
    "longread_dnaseq": {"node": "longread_dnaseq_protocol_help"},
    "methylseq": {"node": "methylseq_protocol_help"},
    "nanopore_covseq": {"node": "nanopore_covseq_protocol_help"},
    "rnaseq": {"node": "rnaseq_protocol_help"},
    "rnaseq_denovo_assembly": {"node": "rnaseq_denovo_assembly_protocol_help"}
  }
} 

Structure:
- variable: name of the variable that needs to be checked
- cases: check value of variable
  - node: depending on the value of the variable, go to id of next node
  
5. input
------------
Asks user to write an input

Example:
{
  "id": "input_directory_path",
  "type": "input",
  "variable": "directory_path",
  "prompt": "Enter the path to the directory you want to run the pipeline in", 
  "next": "store_directory_path"
},

Structure:
- variable: name of variable where the user's input will be stored 
- prompt: ask user to enter value
- next: id of node to enter next 

6. message
------------
Displays a message to the user. Supports variable substitution.

Example:
{
  "id": "protocol_choice_message",
  "type": "message",
  "message": "Your choice of protocol is: {{protocol_name}}",
  "next": "command_help"
}

Structure:
- message: the text shown, variables in {{ }} are replaced with the value 
- next: id of next node

------------------------
List of set_variable
------------------------
From general_guide.JSON:
- pipeline_name: ampliconseq, chipseq, covseq, dnaseq, longread_dnaseq, methylseq, longread_methylseq, nanopore_covseq, rnaseq, 
                  rnaseq_denovo_assembly, rnaseq_light
- protocol_name: chipseq, atacseq, germline_snv, germline_sv, germline_high_cov, somatic_tumor_only, somatic_fastpass,
                 somatic_ensemble, somatic_sv, nanopore, revio, bismark, gembs, dragen, hybrid, default, basecalling, stringtie,
                 variants, cancer, trinity, seq2fun

From command_guide.JSON:
- r_command: -r {{raw_readset_filename}}
- j_command: -j slurm, -j pbs, -j batch
- scheduler_server_name: rorqual, fir, narval, abacus, batch 
- server_in: GENPIPES_INIS/common_ini/{{scheduler_server_name}}.ini
- path_custom_ini: {{raw_path_custom_ini}}
- c_command: -c $GENPIPES_INIS/{{pipeline_name}}/{{pipeline_name}}.base.ini
             -c $GENPIPES_INIS/{{pipeline_name}}/{{pipeline_name}}.base.ini $GENPIPES_INIS/common_ini/{{scheduler_server_name}}.ini
             -c $GENPIPES_INIS/{{pipeline_name}}/{{pipeline_name}}.base.ini $GENPIPES_INIS/{{pipeline_name}}/{{path_custom_ini}}
             -c $GENPIPES_INIS/{{pipeline_name}}/{{pipeline_name}}.base.ini $GENPIPES_INIS/common_ini/{{scheduler_server_name}}.ini $GENPIPES_INIS/{{pipeline_name}}/{{path_custom_ini}}
- genome_config_ini: reference genome config ini if not human GRCh38
    - Mouse: $MUGQIC_INSTALL_HOME/genomes/species/Mus_musculus.mm10/Mus_musculus.mm10.ini
    - Rat: $MUGQIC_INSTALL_HOME/genomes/species/Rattus_norvegicus.Rnor_6.0/Rattus_norvegicus.Rnor_6.0.ini
    - Other (user inputs reference_genome_name and reference_genome_ini_filename in previous question): $MUGQIC_INSTALL_HOME/genomes/species/{{reference_genome_name}}/{{reference_genome_ini_filename}}
- o_command: {empty placeholder if user skips}, -o {{directory_name}}
- d_command: {empty placeholder if user skips}, -d {{design_file_name}}
- p_command: {empty placeholder if user skips}, -p {{pair_file_name}}
- s_command: {empty placeholder if user skips}, -s {{step_range}}
- g_command: -g {{g_filename}}
- final_command: genpipes {{pipeline_name}} {{t_command}} {{c_command}} {{genome_config_ini}} {{r_command}} {{d_command}} {{p_command}} {{j_command}} {{s_command}} {{o_command}} {{g_command}}

From step_guide.JSON:
- step_range: 
  - ampliconseq: 1-6, 8 
  - chipseq chipseq: 1-17, 19-23 
  - chipseq atacseq: 1-18, 20-24 
  - methylseq bismark/hybrid: 1-14, 17-18 
  - methylseq gembs/dragen: 1-16, 19-20 
  - rnaseq stringtie: 1-18, 20-21 
  - rnaseq_denovo_assembly trinity: 1-19, 21-24 
  - rnaseq_denovo_assembly seq2fun: 1-3, 5
  - rnaseq_light: 1-6, 8
  - longread_methylseq: TBD

------------------------------------------------------------------------------------------------------------------------------------
## `general_guide.JSON`
This file contains the general questions that the wizard will ask the user to determine which guide they need help with. 
In cases where the user skips a guide, they will be asked to select their choice of deployment method/pipeline/protocol.

- Deployment guide  
- Pipeline guide  
- Protocol guide  
- Command guide (within this guide, the user can also follow the step guide if needed)

------------------------------------------------------------------------------------------------------------------------------------
## `deployment_guide.JSON`
This file contains the questions that help the user determine the deployment method they want to use to deploy GenPipes

**Deployment method options:**
- `DRAC infrastructure`
- `cloud`
- `container`
- `locally`

------------------------------------------------------------------------------------------------------------------------------------
## `pipeline_guide.JSON`
This file contains the questions that help the user determine the appropriate pipeline based on their dataset and analysis goals.  

**Pipeline options:**
- `ampliconseq`
- `chipseq`
- `covseq`
- `dnaseq`
- `longread_dnaseq`
- `methylseq`
- `longread_methylseq`
- `nanopore_covseq`
- `rnaseq`
- `rnaseq_denovo_assembly`
- `rnaseq_light`

------------------------------------------------------------------------------------------------------------------------------------
## `protocol_guide.JSON`
This file contains the questions used to determine the appropriate protocol based on the dataset and analysis goals.
Note: if ampliconseq/nanopore_covseq/covseq/rnaseq_light then skip question asking user if they need pipeline guide.

**Protocol options:**
- `chipseq` → `chipseq`, `atacseq`  
- `dnaseq` → `germline_snv`, `germline_sv`, `germline_high_cov`, `somatic_tumor_only`, `somatic_fastpass`, 
`somatic_ensemble`, `somatic_sv`  
- `longread_dnaseq` → `nanopore`, `revio`  
- `methylseq` → `bismark`, `gembs`, `dragen`, `hybrid`  
- `nanopore_covseq` → `default`, `basecalling`
- `rnaseq` → `stringtie`, `variants`, `cancer`  
- `rnaseq_denovo_assembly` → `trinity`, `seq2fun`

------------------------------------------------------------------------------------------------------------------------------------
## `command_guide.JSON`
This file contains the questions that the wizard will ask the user to construct the appropriate command based on their pipeline, 
protocol, readset file, job scheduler, design/pair file, directory, steps, etc.

- Pipeline  
- Protocol  
- Readset file  
- Job scheduler  
- Design/pair file  
- Output directory  
- Steps, etc.

------------------------------------------------------------------------------------------------------------------------------------
## `step_guide.JSON`
This file defines the step range to be added to the command based on the pipeline and protocol, 
specifically when the user does not have a design file and wants to run GenPipes. Steps involving the design file must be skipped.

**Step-based options:**
- `ampliconseq`  
- `chipseq` → `chipseq`, `atacseq`  
- `methylseq` → `bismark/hybrid`, `gembs/dragen`  
- `rnaseq` → `stringtie`  
- `rnaseq_denovo_assembly` → `trinity`, `seq2fun`  
- `rnaseq_light`