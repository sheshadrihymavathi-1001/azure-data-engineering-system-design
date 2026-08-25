# Incremental Loading & CDC — Interview Questions

## Beginner

1. What is incremental loading?
2. What is a watermark?
3. Why shouldn't a watermark be updated before target commit?
4. What is CDC?
5. What is idempotency?
6. Why use an overlap window?

## Intermediate

1. Why can a timestamp watermark miss records?
2. Why use a composite watermark?
3. How do you detect deletes without CDC?
4. What happens if a pipeline crashes after target commit but before checkpoint commit?
5. Why is CDC better than timestamp filtering for deletes?
6. What are the limitations of snapshot comparison?

## Advanced

1. How would you process 800M rows without CDC?
2. How would you design hierarchical snapshot comparison?
3. How would you handle data skew?
4. What happens when processing time exceeds the snapshot interval?
5. How do you guarantee idempotent replay?
6. How do you recover when CDC retention expires?
7. How would you migrate from snapshot comparison to CDC?
