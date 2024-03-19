# Troubleshooting for importing *gempyor* into RStudio

## 📖 Preface

Prior to importation into RStudio, *gempyor* must be installed onto your machine. Navigating to your flepiMoP directory, the command ```pip install -e flepimop/gempyor_pkg/``` will install the *gempyor* package with all dependencies. 

The [Quick Start Guide](https://iddynamics.gitbook.io/flepimop/how-to-run/quick-start-guide) section enumerates installation steps in greater detail.

Inference runs of *flepiMoP* require the following packages to be installed in R:

```install.packages(c("readr","sf","lubridate","tidyverse","gridExtra","reticulate","truncnorm","xts","ggfortify","flextable","doParallel","foreach","optparse","arrow","devtools","cowplot","ggraph"))```

Though, for troubleshooting importation, just the package ```reticulate``` will be required.

## ➡️ Importation

*gempyor* can be initially imported into RStudio as follows:

```gempyor <- reticulate::import("gempyor")```

## 🚧 Error Troubleshooting

#### If you run into errors, there are some steps you can take (or may need to take) to help *gempyor* import correctly.

1. Configure which version of Python to use
    - The ```reticulate::use_python()``` function can be used to do this
    - You will pass the pathway to your Python binary (the same one you used to install *gempyor* onto your machine) into ```use_python()```
        - You can retrieve this pathway by navigating to your *flepiMoP* directory and running the command line command ```which python```
        - Pass this as a string (```""```) into ```use_python()```
    - In RStudio, simply running ```py_config()``` will provide more information about the version of Python currently being used by ```reticulate```


2. Ensure that Python versions match
    - ```retiuclate``` could be attempting to import *gempyor* using a version of Python that is different from the version you used to install *gempyor* onto your machine
    - In RStudio, simply running ```py_config()``` will provide more information about the version of Python currently being used by ```reticulate```
    -  This version should match what is returned when you run the command line command ```python3 --version``` (when in your flepiMoP directory).

#### After completing steps 1 and 2, retry initializing ```gempyor <- reticulate::import("gempyor")```. If you continue to have roadblocks (**especially** if you have recently made changes in your flepiMoP directory), closing your RStudio session ad re-opening your file could mitigate these issues. 
