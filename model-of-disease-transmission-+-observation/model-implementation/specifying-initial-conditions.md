# Specifying initial conditions

## `seeding` section

There are two different seeding methods: 1) based on air importation (FolderDraw) and 2) based on earliest identified cases (PoissonDistributed)

FolderDraw is required if the importation section is present and requires `folder_path`. Otherwise, put PoissonDistributed, which requires `lambda_file`.

| Config Item | Required?                | Type/Format                   | Description |
|-------------|--------------------------|-------------------------------|-------------|
| method      | **required**             | "FolderDraw" or "PoissonDistributed" | |
| folder_path | required for FolderDraw  | path to folder                | |
| lambda_file | required for PoissonDistributed | path to file           | |
| delay_incidC| optional for PoissonDistributed | numeric | Assumption for number of days delay between infection and case confirmation for seeding with the PoissonDistributed method. Default is 5 days. |
| ratio_incidC | optional for PoissonDistributed | numeric | Assumption for ratio of infections to confirmed cases for seeding with the PoissonDistributed method. Default is 10 infections per confirmed case. | |
| casedata_file | required for non-US locations | path to the data file from which the seeding setup file will be created | | 

If using the `importation` section of the config and the air importation model:
```
seeding:
  method: FolderDraw
  folder_path: importation/HI/
```
or if seeding according to the earliest identified cases:
```
seeding:
  method: PoissonDistributed
  lambda_file: data/HI/seeding.csv
  delay_incidC: 5
  ratio_incidC: 10
```