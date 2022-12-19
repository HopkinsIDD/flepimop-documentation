# Useful commands

TODO: add how to run test, and everything

{% hint style="danger" %}
Don't paste them if you don't know what they do
{% endhint %}

### Filepaths structure

in configs with a setup `name: USA`

```
model_output/{FileType}/{Prefix}{Index}.{run_id}.{FileType}.{Extension}
                           ^ 
                          setup name(USA)/scenario(inference/med)/run_id/{Inference stuff}
                                                                           ^ global/{final, intermediate}/slot#.
```

where, eg:

* the index is `1`
* the run\_id is `2021.12.14.23:56:12.CET`
* the prefix is `USA/inference/med/2021.12.14.23:56:12.CET/global/intermediate/000000001.`

### Steps to first local run

```bash
export COVID_PATH=$(pwd)/COVIDScenarioPipeline
export DATA_PATH=$(pwd)/COVID19_USA
conda activate covidSP
cd $COVID_PATH
Rscript local_install.R
pip install --no-deps -e gempyor_pkg # before: python setup.py develop --no-deps
git lfs install
git lfs pull
export CENSUS_API_KEY=YOUR_KEY
cd $DATA_PATH
git restore data/
export CONFIG_PATH=config_smh_r11_optsev_highie_base_deathscases_blk1.yml
Rscript $COVID_PATH/R/scripts/build_US_setup.R
Rscript $COVID_PATH/R/scripts/create_seeding.R
Rscript $COVID_PATH/R/scripts/full_filter.R -j 1 -n 1 -k 1
```

where:

* $$n$$ is slots
* $$j$$ is core
* $$k$$ is iteration per slot

### Launch the docker locally

```bash
docker pull hopkinsidd/covidscenariopipeline:latest-dev
docker run -it -v "$(pwd)":/home/app/covidsp hopkinsidd/covidscenariopipeline:latest-dev
```

### Pipeline git-fu (dealing with the commute\_data)

because a big file get changed and added automatically. Since Git 2.13 (Q2 2017), you can stash individual files, with [_git stash push_](https://git-scm.com/docs/git-stash#git-stash-push-p--patch-k--no-keep-index-u--include-untracked-a--all-q--quiet-m--messageltmessagegt--ltpathspecgt82308203). One of these should work.

```bash
git restore --staged sample_data/united-states-commutes/commute_data.csv
git stash push sample_data/united-states-commutes/commute_data.csv
git reset sample_data/united-states-commutes/commute_data.csv
```
