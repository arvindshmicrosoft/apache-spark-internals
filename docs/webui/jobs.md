# Jobs Tab

*Jobs* tab in index.md[web UI] shows [status of all Spark jobs](AllJobsPage.md) in a Spark application (SparkContext.md[]).

.Jobs Tab in Web UI
image::spark-webui-jobs.png[align="center"]

Jobs tab is available under `/jobs` URL (e.g. http://localhost:4040/jobs).

.Event Timeline in Jobs Tab
image::spark-webui-jobs-event-timeline.png[align="center"]

The Jobs tab consists of two pages, i.e. [All Jobs](AllJobsPage.md) and <<JobPage, Details for Job>> pages.

Internally, the Jobs tab is represented by [JobsTab](JobsTab.md).

== [[JobPage]] Details for Job -- `JobPage` Page

When you click a job in [AllJobsPage](AllJobsPage.md), you see the *Details for Job* page.

.Details for Job Page
image::spark-webui-jobs-details-for-job.png[align="center"]

`JobPage` is a spark-webui-WebUIPage.md[WebUIPage] that shows statistics and stage list for a given job.

Details for Job page is registered under `/job` URL, i.e. `http://localhost:4040/jobs/job/?id=0` and accepts one mandatory `id` request parameter as a job identifier.

When a job id is not found, you should see "No information to display for job ID" message.

."No information to display for job" in Details for Job Page
image::spark-webui-jobs-details-for-job-no-job.png[align="center"]

`JobPage` displays the job's status, group (if available), and the stages per state: active, pending, completed, skipped, and failed.

NOTE: A job can be in a running, succeeded, failed or unknown state.

.Details for Job Page with Active and Pending Stages
image::spark-webui-jobs-details-for-job-active-pending-stages.png[align="center"]

.Details for Job Page with Four Stages
image::spark-webui-jobs-details-for-job-four-stages.png[align="center"]
