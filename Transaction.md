### Nature of transaction
Usually transactions are used to change and modify data
However, it is perfectly normal to have a read only transaction
example, you want to generate a report and you want to get consistent data
snapshot based at the time of transaction start, so when the data changes during the report generation, you will not get inconsistent data