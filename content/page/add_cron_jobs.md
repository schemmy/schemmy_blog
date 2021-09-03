---
title: "Add scheduled jobs via Cron on AWS EC2"
date: 2020-04-02T21:01:50-07:00
# draft: true
tags: ["linux", "memo"]
---

By using Cron, we can schedule the running of scripts at a specific date and time. However, it took me a while to figure out how to make it work on AWS EC2 to schedule a python task. Below are steps.

1). Need to add ```conda activate env_name``` in bash file, so that conda environment is activated automatically before Cron runs.

2). Create Cron jobs by ```crontab -e```. After setting up the scheduled time following [syntax of Crontab](https://docs.oracle.com/cd/E19455-01/805-7229/sysrescron-62861/index.html), we need to first ```cd``` to the path of python script. Finally the command should be:

```30 13 * * 1,2,3,4,5  cd /absolute/path/to/script/ && $(which python) /absolute/path/to/script/main.py >> ~/cron.log 2>&1```

Notice that, ```cd``` command cannot be ignored, even though the absolute path is still needed in the last step. Also, step 1) is also necessary. It won't run if ```$(which python)``` is replaced by the absolute path of the python location.