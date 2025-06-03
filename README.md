# Query-for-Power-bi-to-thingsboard

let
    // 1) Your JWT token (including "Bearer "):
    jwtToken = "Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhaHRpc2hhbWFAbGVhbnNvbC5jYSIsInVzZXJJZCI6IjUzN2NjYmUwLWJlMGItMTFlZi1iNWE4LWVkMWFlZDlhNjUxZiIsInNjb3BlcyI6WyJURU5BTlRfQURNSU4iXSwic2Vzc2lvbklkIjoiZGViMGIxOWMtMTVhNC00YTZkLThhYWQtNGYwMDFiYmJjMzU0IiwiZXhwIjoxNzUwNzQwNjkxLCJpc3MiOiJ0aGluZ3Nib2FyZC5pbyIsImlhdCI6MTc0ODk0MDY5MSwiZmlyc3ROYW1lIjoiQWh0aXNoYW0gQSIsImVuYWJsZWQiOnRydWUsInByaXZhY3lQb2xpY3lBY2NlcHRlZCI6dHJ1ZSwiaXNQdWJsaWMiOmZhbHNlLCJ0ZW5hbnRJZCI6IjUxZjgyN2IwLWJlMGItMTFlZi1iNWE4LWVkMWFlZDlhNjUxZiIsImN1c3RvbWVySWQiOiIxMzgxNDAwMC0xZGQyLTExYjItODA4MC04MDgwODA4MDgwODAifQ.TP8UQRhJQyFhdCOrgK4_kB0wYR4X3bQYB9VhgG3DKv78Aem_aA6Lcx45uSG3cEc8ZHU9f2QJUh6ebcmriNijdg",

    // 2) REST endpoint URL with YOUR host and device ID, requesting only the "data" key:
    endpointUrl =
        "http://demo.thingsboard.io/api/plugins/telemetry/DEVICE/5c4f9e30-3fa9-11f0-9664-ff13eccb47ff/values/timeseries?keys=voltage_a,ct_ratio,frequency,device_id",

    // 3) Call ThingsBoard’s REST API:
    rawResponse = Web.Contents(
        endpointUrl,
        [
            Headers = [
                #"X-Authorization" = jwtToken,
                #"Content-Type"   = "application/json"
            ]
        ]
    ),
    parsedJson = Json.Document(rawResponse),

    // 4) Turn the top‐level record (only has one field "data") into a table:
    asTable = Record.ToTable(parsedJson),

    // 5) Expand the list of {ts, value} records into separate rows:
    expanded = Table.ExpandListColumn(asTable, "Value"),
    expandedRecords = Table.ExpandRecordColumn(
        expanded,
        "Value",
        {"ts", "value"},
        {"timestamp", "value"}
    ),

    // 6) (Optional) Convert “timestamp” from Unix‐ms to a true DateTime:
    toDateTime =
        Table.TransformColumns(
            expandedRecords,
            {{"timestamp", each #datetime(1970,1,1,0,0,0) + #duration(0,0,0,0, _/1000), type datetime}}
        ),
    #"Removed Columns" = Table.RemoveColumns(toDateTime,{"timestamp"})
in
    #"Removed Columns"
