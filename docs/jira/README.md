# User Guide for Jira
https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/

# Running tests
## Pre-requisites
* Working Jira Software of supported version ([toolkit README](../../README.md) for a list of supported Jira versions) with users, issues, projects, and boards, etc.
* Client machine with 4 CPUs and 16 GBs of RAM to run the Toolkit.
* Virtual environment with Python3.6+ and bzt installed. See the root [toolkit README](../../README.md) file for more details.

If you need performance testing results at a production level, follow instructions described 
in the official User Guide to set up Jira DC with the corresponding dataset.
For spiking, testing, or developing, your local Jira instance would work well.

## Step 1: Update jira.yml
* `application_hostname`: test jira hostname (without http).
* `application_protocol`: http or https.
* `application_port`: 80 (for http) or 443 (for https), 8080, 2990 or your instance-specific port.
* `secure`: True or False. Default value is True. Set False to allow insecure connections, e.g. when using self-signed SSL certificate.
* `application_postfix`: it is empty by default; e.g., /jira for url like this http://localhost:2990/jira.
* `admin_login`: jira admin user name (after restoring dataset from SQL dump, the admin user name is: admin).
* `admin_password`: jira admin user password (after restoring dataset from SQL dump, the admin user password is: admin) .
* `load_executor`: executor for load tests. Valid options are [jmeter](https://jmeter.apache.org/) (default) or [locust](https://locust.io/).
* `concurrency`: `200` - number of concurrent users for JMeter scenario.
* `test_duration`: `45m` - duration of test execution.
* `ramp-up`: `3m` - amount of time it will take JMeter or Locust to add all test users to test execution.
* `total_actions_per_hour`: `54500` - number of total JMeter/Locust actions per hour.
* `WEBDRIVER_VISIBLE`: visibility of Chrome browser during selenium execution (False is by default).

## Step 2: Run tests
Run Taurus.
```
bzt jira.yml
```

## Results
Results are located in the `resutls/jira/YY-MM-DD-hh-mm-ss` directory:
* `bzt.log` - log of bzt run
* `error_artifacts` - folder with screenshots and HTMLs of Selenium fails
* `jmeter.err` - JMeter errors log
* `locust.err` - Locust errors log
* `kpi.jtl` - JMeter raw data
* `pytest.out` - detailed log of Selenium execution, including stacktraces of Selenium fails
* `selenium.jtl` - Selenium raw data
* `results.csv` - consolidated results of execution
* `resutls_summary.log` - detailed summary of the run. Make sure that overall run status is `OK` before moving to the 
next steps.


# Useful information

## Changing performance workload for JMeter and Locust
The [jira.yml](../../app/jira.yml) has a `action_name` fields in `env` section with percentage for each action. You can change values from 0 to 100 to increase/decrease execution frequency of certain actions. 
The percentages must add up to 100, if you want to ensure the performance script maintains 
throughput defined in `total_actions_per_hr`. The default load simulates an enterprise scale load of 54500 user transactions per hour at 200 concurrency.

To simulate a load of medium-sized customers, `total_actions_per_hr` and `concurrency` can be reduced to 14000 transactions and 70 users. This can be further halved for a small customer.

## JMeter
### Opening JMeter scripts
JMeter is written in XML and requires the JMeter GUI to view and make changes. You can launch JMeter GUI by running the `~/.bzt/jmeter-taurus/<jmeter_version>/bin/jmeter` command. 
Be sure to run this command inside the `app` directory. The main [jira.jmx](../../app/jmeter/jira.jmx) file contains the relative path to other scripts and will throw errors if run elsewhere. 

### Debugging JMeter scripts
1. Open JMeter GUI from `jira` directory by running the `~/.bzt/jmeter-taurus/<jmeter_version>/bin/jmeter` command. 
1. Right-click `Test Plan` > `Add` > `Listener` > `View Results Tree`. 
1. On the `View Results Tree` page, click the `Browse` button and open `error.jtl` from `app/results/jira/YY-MM-DD-hh-mm-ss` folder.

From this view, you can click on any failed action and see the request and response data in appropriate tabs.

In addition, you can run and monitor JMeter test real-time with GUI.
1. Launch the test with GUI by running `bzt jira.yml -gui`.
1. Right-click `Test Plan` > `Add` > `Listener` > `View Results Tree`. 
1. Click the start button to start running the test.

### Run one JMeter action
#### Option 1: Run one JMeter action via GUI
1. Open JMeter GUI from `app` directory by running the `~/.bzt/jmeter-taurus/<jmeter_version>/bin/jmeter` command. 
1. Go to `File` > `Open`, and then open `jmeter/jira.jmx`.
1. In the `Global Variables` section, add correct Jira hostname, port, protocol, and postfix (if required).
1. In `Jira` > `load profile`, set `perc_desired_action` to 100.
1. Enable `View Results Tree` controller.
1. Run JMeter.
1. `View Results Tree` controller will have all details for every request and corresponding response..

#### Option 2: Run one JMeter action via bzt
1. In [jira.yml](../../app/jira.yml), set `perc_desired_action` to 100 and all other perc_* to 0.
1. Run `bzt jira.yml`.


## Locust
### Debugging Locust scripts
Detailed log of Locust executor is located in the `results/jira/YY-MM-DD-hh-mm-ss/locust.log` file. Locust errors and stacktrace are located in the `results/jira/YY-MM-DD-hh-mm-ss/locust.err` file.

Additional debug information could be enabled by setting `verbose` flag to `true` in `jira.yml` configuration file. To add log message use `logger.locust_info('your INFO message')` string in the code.
### Running Locust tests locally without the Performance Toolkit
#### Start locust UI mode
1. Activate virualenv for the Performance Toolkit.
1. Navigate to `app` directory and execute command `locust --locustfile locustio/jira/locustfile.py`. 
1. Open your browser, navigate to `localhost:8089`.  
1. Enter `Number of total users to simulate` (`1` is recommended value for debug purpose)  
1. Enter `Hatch rate (users spawned/secods)` 
1. Press `Start spawning` button.

#### Start Locust console mode
1. Activate virualenv for the Performance Toolkit.
1. Navigate to `app` and execute command `locust --no-web --locustfile locustio/jira/locustfile.py --clients N --hatch-rate R`, where `N` is the number of total users to simulate and `R` is the hatch rate.  

Full logs of local run you can find in the `results/jira/YY-MM-DD-hh-mm-ss_local/` directory.

To execute one locust action, navigate to `jira.yml` and set percentage value `100` to the action you would like to run separately, set percentage value `0` to all other actions.


## Selenium
### Debugging Selenium scripts
Detailed log and stacktrace of Selenium PyTest fails are located in the `results/jira/YY-MM-DD-hh-mm-ss/pytest.out` file. 

Also, screenshots and HTMLs of Selenium fails are stared in the `results/jira/YY-MM-DD-hh-mm-ss/error_artifacts` folder. 

### Running Selenium tests with Browser GUI
In [jira.yml](../../app/jira.yml) file, set the `WEBDRIVER_VISIBLE: True`.


### Running Selenium tests locally without the Performance Toolkit
1. Activate virualenv for the Performance Toolkit.
1. Navigate to the selenium folder using the `cd app/selenium_ui` command. 
1. In [jira.yml](../../app/jira.yml) file, set the `WEBDRIVER_VISIBLE: True`.
1. Run all Selenium PyTest tests with the `pytest jira-ui.py` command.
1. To run one Selenium PyTest test (e.g., `test_1_selenium_view_issue`), execute the first login test and the required one with this command:

`pytest jira-ui.py::test_0_selenium_a_login jira-ui.py::test_1_selenium_view_issue`.


### Comparing different runs
Navigate to the `reports_generation` folder and follow README.md instructions to generate side-by-side comparison charts.
