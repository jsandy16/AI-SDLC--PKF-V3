# sas_restructure: open questions

The run stopped: these questions have no safe default. Answer them in a YAML file (question id or text → answer) and run `pkf create sas_restructure --requirement <file> --answers <answers.yaml>`; the answers are added to the requirement, which is analysed again. For the analysis questions, `--assume-all` accepts the proposed assumptions instead.

## ENV-sas_bucket

What is the development value of `sas_bucket` (The environment's SAS S3 bucket, used as both source and destination)?

Answers file template:

```yaml
ENV-sas_bucket: ""
```
