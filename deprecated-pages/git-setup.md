# Git Setup

SETUP

* Basically we are putting one repo inside the other one and treating them as independent. You push and pull code in them independently and you need to make sure they are both up to date or at the specific commit that you need

`git clone https://github.com/HopkinsIDD/COVID19_California.git` (or other spatial repo)

`cd COVID19_California`

`git clone https://github.com/HopkinsIDD/COVIDScenarioPipeline.git`

`cd COVIDScenarioPipeline`

`git checkout origin/dataseed -b dataseed` ## assuming we're still in the dataseed branch

CHECKING STATUS Navigate to the correct repository directory. `git status` _Note that you will not see modifications to tracked files in the repository COVIDScenarioPipeline if you are in the spatial repo directory._

PUSH Navigate to the correct repository directory.

`git checkout branchname`

`git add newfile`

`git commit`

`git push`

PULL Navigate to the correct repository directory. Make sure you are up to date (add new files and commit them)

`git checkout branchname`

`git pull`
