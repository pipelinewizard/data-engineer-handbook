Day 2 - Slowly Changing Dimensions
Key Points
- Slowly Changing Dimensions - An attribute that drift over time.
- other Dimensions dont change - Like your birthday
- Idempotent - The ability for your data pipeline to produce the same results whether its running in production or Backfill

Slide 1 - Idempotent Pipelines are Critical
Idempotent - Denoting an element of a set which is unchanged in value when multiplied or otherwise operated on by itself

Slide 2 - Pipelines should produce the same results
- If you have the appropriate inputs - the pipeline should producce the same results, no matter the time frame or frequency (Hour, Day, Week, Month, Year)
- What happens if you dont follow this principle? ALot of pain suffering from downstream users dealing with data discrepancy.

Slide 3 - Why is troubleshooting non-idempotent pipelines hard?
- The pipeline doesnt fail, it just produces non-reproducible data (SILENT FAILURE) (Data Analyst - Why dont these numbers match!)
- The data engineer could be dependeing on a non-idempotent data pipeline

Slide 4 - What can make a pipeline not idempotent?
- INSERT INTO w/o TRUNCATE - this causes you to insert duplicate data into the table which thenc auses the pipeline to not get the same results each time its ran.
    - its better to use MERGE or INSERT OVERWRITE
        - MERGE will merge the new data with the old data recognizing matching values and not create duplicate values
        - INSERT OVERWRITE - Instead of matchinga ll the records, you just overwrite the partition.
- Using Start_date > without a corresponding end date - Because each day that passes is adding a new day of data, which means the everytime the pipeline is ran it does not produce the same result.
    - You could also create "Out of Memory" issues by processing unbounded limitless data
    - ALWAYS PUT AN END DATE to create a window or else it will be alot of data 
- Not using a full set of partition sensors
    - This means the pipline runs before all the inputs are ready, which creates an issue in production because when you backfill Production and Backfill data dont match.
- Not using 'depends_on_past' for cumalative pipelines (aka ariflow or sequential processing)
    - If you have a pipeline that depends on yesterdays data, where were outer joining Yesterday and Today's data - that means that the pipeline cant run in parrallel (Yesterday, then tomorrow, Then the Next Day)
    You dont want to process an unbounded amount of data without a limit
- Relying on the latest partition of an upstream dataset not properly modeled SCD table
If you are backfilling data and you have a properly modeled SCD then you can rely on the latest data.

Slide 5 - The Pains of not having idempotrent pipelines
- If pipeline is not idempotnent backfill and production data will not be the same.
- Hard to troubleshoot bugs
- Unit Testing cant replicate the production behavior

Slide 6 - Should you model as Slowly Changing Dimensions?
- Slowly Changing Dimension: A Dimension that changes over time.(e.g., Age, Country you live, Iphone User not Android User)
- Rapidly Changing Dimension - Heart Rate Minute to Minute
- The slower the dimension changes, the better.
- What are the 3emporal dimensionsal data modeling options?
    - Latest Snapshot - Instead of modeling day by day you have the current value - the problem if you have a slowly changing dimension the pipeline is non-idempotent?
    - Daily/Monthly/Yearly Snapshot - Instead of the latest snapshot you use daily, and it pulls the dimension on that day for that particular recod
    - SCD - Collapsing the dimension into 1 row instead of a row for each day (Compression is key)

Slide 7 - WHy do dimensions change?
- Preferences change (e.g., iphone or android, team dog or team cat, migrates from US to another country)

Slide 8 - HGow can you model dimensions that change\- 3 Ways to model them
- Singular snapshots - These are not idempotent because the current dimension could not be reflective of the current data.
    - NEVER DO This
- Daily partioned snapshot - very simple, everyday we record a value for the dimensions
- SCD Type 0 - If you are sure your dimension wont change (Table has id and value)
- SCD Type 1 - You only care about the latest value (Never use this it makes pipelines non-idempotent) Important for analyticv jobs
- SCD Type 2 - Air BNB calls the Gold Standard (You have a start date and an end date that shows when dimension changes)
    - Current Value has an end date that is null or far out into the future.
    - More than 1 row per dimension (Need to be more careful about filteringt on time)
    - There is a lot of time where there is another column called 'is_current'
- SCD Type 3 - You only care about original and current value
    - You only have 1 row per dimension
    - Drawback: You lose the history in between original and current
    - Non-idempotent

Slide 8 - Which types are idempotent?
- Type 0 and Type 2 are idempotent
    - Type 0 because values are unchanging
    - Type 2 is idempotent but you have to filter dimension tables on start and end dataset
    - Type 1 is NOT idempotent, if you backfill with this dataset, you'll get the dimension as it is now, not as it was then.
    - Type 3 is NOT idepotent - if you backfill the dataset its impossible to know when to pick original vs. current

Slide 9 - SCD2 Loading
- You can load the entire history in one query
    - Inefficiwent but nimble
    - 1 query and you're done
- Incrementally load the data after the previous SCD is generated
    - Has the same "depends_on_past" constraint
    - Efficienet but cumbersome`
- Not every pipeline has to be perfect. You are wasting time on marginal value when you could be looking at bigger fish.


Questions:
What does backfilling a pipeline mean? I think it means updating the non-production datasets (DEV, TEST) - Or does it mean leading older data into the pipeline.
What are partion sensors?
Still confused on depends_on_past and how it enhances idempotency?
How does unit testing work?