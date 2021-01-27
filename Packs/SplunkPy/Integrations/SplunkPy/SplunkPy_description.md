### Use the SplunkPy integration to fetch incidents from Splunk ES, and query results by SID.

### Enriching fetch mechanism
***
Starting from version 2.0.0 of SplunkPy Pack, we allow the option to enrich fetched notables.
There are 3 possible enrichment types:
1. Drilldown search enrichment: fetches the drilldown search configured by the user in the rule name that triggered the notable and performs this search. The results are stored in the context of the incidents under the `Drilldown` field.
2. Asset search enrichment: Runs the following query `| inputlookup append=T asset_lookup_by_str where asset=$ASSETS_VALUE | inputlookup append=t asset_lookup_by_cidr where asset=$ASSETS_VALUE | rename _key as asset_id | stats values(*) as * by asset_id` where the `$ASSETS_VALUE` is replaced with the `src`, `dest`, `src_ip` & `dst_ip` from the fetched notable. The results are stored in the context of the incidents under the `Asset` field.
3. Identity search enrichment: Runs the following query `| inputlookup identity_lookup_expanded where identity=$IDENTITY_VALUE` where the `$IDENTITY_VALUE` is replaced with the `user` & `src_user` from the fetched notable. The results are stored in the context of the incidents under the `Identity` field.

#### How to configure
1. Configure the integration to fetch incidents.
2. Go to the `Enrichment Types` parameter and select the enrichment types you want to enrich each fetched notable. If none are selected, the integration will fetch notables as usual (without enrichment).
3. Go to the `Enrichment Timeout (Minutes)` parameter and select the timeout for each enrichment. When the selected timeout was reached, notable events that were not enriched will be saved without the enrichment.
4. Go to the `Number of Events Per Enrichment Type` parameter and select the maximal amount of events to fetch per enrichment type.

#### Knowing the status of an enrichment
Each enriched incident **can** containt the following fields in the incident context:
- `successful_drilldown_enrichment`: whether the drilldown enrichment was successful or not.
- `successful_asset_enrichment`: whether the asset enrichment was successful or not.
- `successful_identity_enrichment`: whether the identity enrichment was successful or not.
- `successful_enrichment`: whether the whole enrichment for the incident was successful or not. Note that if only part of all enrichments was successful this field will be set to True, i.e. no need for all of the enrichments to succeed to consider the full enrichment as successful. (This field will always be shown)

#### Resetting the enriching fetch mechanism
- Run the `splunk-reset-enriching-fetch-mechanism` and the mechanism will be reset to first configuration (No need to reset the `Last Run` object)

#### Limitations
- As the enrichment process is asynchronous, fetching enriched incidents takes longer. The integration was tested with 20+ fetched notables that were enriched after approximately ~4min.
- If you with to configure a mapper, wait for the integration to perform a first fetch, this is to make the fetch mechanism logic stable. 

