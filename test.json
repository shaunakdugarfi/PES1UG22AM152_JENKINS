config {
    type: "incremental",
    database: "common-tech-434709",
    schema: "dataview",
    name: "comms_view",
    description: "View for communications metrics across different channels and services",
    uniqueKey: ["ACTOR_ID", "COMMS_DATE", "COMMS_L1", "COMMS_L2"],
    assertions: {
        nonNull: ["ACTOR_ID", "COMMS_DATE", "COMMS_L1", "COMMS_L2", "STATUS"]
    },
    bigquery: {
        partitionBy: {
            field: "COMMS_DATE",
            type: "DAY"
        },
        clusterBy: ["ACTOR_ID", "COMMS_L1", "COMMS_L2"]
    },
    tags: ["type: incremental", "schema: dataview", "area: comms"],
    columns: {
        COMMS_DATE: "Date of the communication event",
        SERVICE: "Service or flow associated with the communication",
        STATUS: "Status of the communication",
        COMMS_L1: "Primary communication category",
        COMMS_L2: "Secondary communication category",
        CAMPAIGN_NAME: "Name of the campaign",
        TREATMENT_ID: "Identifier for the treatment group",
        MASKED_ID_CN: "Masked identifier for the user",
        ACTOR_ID: "Unique identifier for the user",
        PNDISPLAYED_FI_SERVER: "Number of push notifications displayed (FI Server)",
        PNDISMISSED_FI_SERVER: "Number of push notifications dismissed (FI Server)",
        PNCLICKED_FI_SERVER: "Number of push notifications clicked (FI Server)",
        INAPPDISPLAYED_FI_SERVER: "Number of in-app notifications displayed (FI Server)",
        INAPPDISMISSED_FI_SERVER: "Number of in-app notifications dismissed (FI Server)",
        INAPPCLICKED_FI_SERVER: "Number of in-app notifications clicked (FI Server)",
        INAPPDISPLAYED_INAPP_BANNER: "Number of in-app banners displayed",
        INAPPDISMISSED_INAPP_BANNER: "Number of in-app banners dismissed",
        INAPPCLICKED_INAPP_BANNER: "Number of in-app banners clicked",
        PNDISPLAYED_PINPOINT: "Number of push notifications displayed (Pinpoint)",
        PNDISMISSED_PINPOINT: "Number of push notifications dismissed (Pinpoint)",
        PNCLICKED_PINPOINT: "Number of push notifications clicked (Pinpoint)",
        ROW_UPDATED_TIME: "Timestamp when the row was last updated"
    }
}


/* Set date parameters and drop existing table */
pre_operations {
    DECLARE start_time_param TIMESTAMP DEFAULT (
    ${when(incremental(),
    `SELECT DATE_SUB((SELECT max(row_updated_time) from ${self()}), INTERVAL 2 DAY)`,
    `SELECT TIMESTAMP('2021-02-12')`)}
  );
  
  DECLARE end_time_param TIMESTAMP DEFAULT (
    SELECT CURRENT_TIMESTAMP()
  );

}

WITH base_data AS (
    SELECT 
        *,
        CASE 
            WHEN LOWER(CAMPAIGN_NAME) LIKE '%preapproved%' OR LOWER(CAMPAIGN_NAME) LIKE '%personalloan%' THEN 'PERSONAL LOAN FLOW'
            WHEN LOWER(CAMPAIGN_NAME) LIKE '%add_money%' THEN 'ADD FUNDS BE FLOW'
            WHEN (LOWER(CAMPAIGN_NAME) LIKE '%jump%' OR LOWER(CAMPAIGN_NAME) LIKE '%p2p%') THEN 'JUMP FLOW'
            WHEN LOWER(CAMPAIGN_NAME) LIKE '%salary_never_started_registration%' THEN 'SALARY NEVER REGISTERED FLOW'
            WHEN LOWER(CAMPAIGN_NAME) LIKE '%salary_started_registration_not_completed%' THEN 'SALARY STARTED BUT REG NOT COMP FLOW'
            WHEN LOWER(CAMPAIGN_NAME) LIKE '%salary_registration_completed%' THEN 'SALARY REG COMP FLOW'
            WHEN LOWER(CAMPAIGN_NAME) LIKE '%salary_active%' THEN 'SALARY REG COMP FLOW'
            WHEN (LOWER(CAMPAIGN_NAME) LIKE '%upi%' OR LOWER(CAMPAIGN_NAME) LIKE '%upioffer%') THEN 'PAY FLOW'
            WHEN (LOWER(CAMPAIGN_NAME) LIKE '%creditanalyser%' OR LOWER(CAMPAIGN_NAME) LIKE '%categoryanalyser%' 
                 OR LOWER(CAMPAIGN_NAME) LIKE '%spendsanalyser%' OR LOWER(CAMPAIGN_NAME) LIKE '%mfanalyser%'
                 OR LOWER(CAMPAIGN_NAME) LIKE '%timeanalyser%' OR LOWER(CAMPAIGN_NAME) LIKE '%creditanalyzer%' 
                 OR LOWER(CAMPAIGN_NAME) LIKE '%categoryanalyzer%' OR LOWER(CAMPAIGN_NAME) LIKE '%spendsanalyzer%'
                 OR LOWER(CAMPAIGN_NAME) LIKE '%mfanalyzer%' OR LOWER(CAMPAIGN_NAME) LIKE '%timeanalyzer%') THEN 'ANALYSER FLOW'
            ELSE service 
        END AS service_mod,
        CASE 
            WHEN LOWER(comms_l1) LIKE '%pn system tray%' THEN 'PUSH NOTIFICATION'
            ELSE comms_l2 
        END AS comms_l2_mod
    FROM 
        `common-tech-434709.datamart.comms_backend`
    WHERE 
        row_updated_time >= start_time_param
        AND row_updated_time < end_time_param
)

SELECT
    comms_date AS COMMS_DATE,
    service_mod AS SERVICE,
    status AS STATUS,
    comms_l1 AS COMMS_L1,
    comms_l2_mod AS COMMS_L2,
    campaign_name AS CAMPAIGN_NAME,
    treatment_id AS TREATMENT_ID,
    masked_id_cn AS MASKED_ID_CN,
    actor_id AS ACTOR_ID,
    pndisplayed_fi_server AS PNDISPLAYED_FI_SERVER,
    pndismissed_fi_server AS PNDISMISSED_FI_SERVER,
    pnclicked_fi_server AS PNCLICKED_FI_SERVER,
    inappdisplayed_fi_server AS INAPPDISPLAYED_FI_SERVER,
    inappdismissed_fi_server AS INAPPDISMISSED_FI_SERVER,
    inappclicked_fi_server AS INAPPCLICKED_FI_SERVER,
    inappdisplayed_inapp_banner AS INAPPDISPLAYED_INAPP_BANNER,
    inappdismissed_inapp_banner AS INAPPDISMISSED_INAPP_BANNER,
    inappclicked_inapp_banner AS INAPPCLICKED_INAPP_BANNER,
    pndisplayed_pinpoint AS PNDISPLAYED_PINPOINT,
    pndismissed_pinpoint AS PNDISMISSED_PINPOINT,
    pnclicked_pinpoint AS PNCLICKED_PINPOINT,
    CURRENT_TIMESTAMP() as ROW_UPDATED_TIME
FROM 
    base_data
WHERE 
    COMMS_DATE >= '2024-01-07'
