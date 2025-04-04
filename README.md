# 🏭 Average Time of Process per Machine

## 🎯 Problem Description

We have a factory website that tracks **machine activity logs**. The activity data is stored in the following table:

---

### 📋 Activity Table

| Column Name    | Type                  |
|----------------|-----------------------|
| machine_id     | int                   |
| process_id     | int                   |
| activity_type  | enum('start', 'end')  |
| timestamp      | float                 |

- **machine_id**: ID of the machine.
- **process_id**: ID of the process being executed on that machine.
- **activity_type**: Describes whether the process was `'start'` or `'end'`.
- **timestamp**: Time in seconds when the activity happened.

📌 The **primary key** is a combination of `(machine_id, process_id, activity_type)`.

🧠 It's guaranteed:
- A process will always have one `start` and one `end`.
- The `start` timestamp is always **before** the `end` timestamp.

---

## ❗ Goal

We need to calculate:
- The total processing time for each machine.
- Divide that by the number of processes it ran.
- Round the result to **3 decimal places**.

---

## ✅ Expected Output Format

| machine_id | processing_time |
|------------|-----------------|
| int        | float (3 decimals) |

---

## 🧠 Example

### Input:

| machine_id | process_id | activity_type | timestamp |
|------------|------------|----------------|-----------|
| 0          | 0          | start          | 0.712     |
| 0          | 0          | end            | 1.520     |
| 0          | 1          | start          | 3.140     |
| 0          | 1          | end            | 4.120     |
| 1          | 0          | start          | 0.550     |
| 1          | 0          | end            | 1.550     |
| 1          | 1          | start          | 0.430     |
| 1          | 1          | end            | 1.420     |
| 2          | 0          | start          | 4.100     |
| 2          | 0          | end            | 4.512     |
| 2          | 1          | start          | 2.500     |
| 2          | 1          | end            | 5.000     |

### Output:

| machine_id | processing_time |
|------------|-----------------|
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |

---

## 🔍 Explanation

**Machine 0**
- (1.520 - 0.712) + (4.120 - 3.140) = 0.808 + 0.980 = 1.788
- Average: 1.788 / 2 = **0.894**

**Machine 1**
- (1.550 - 0.550) + (1.420 - 0.430) = 1.000 + 0.990 = 1.990
- Average: 1.990 / 2 = **0.995**

**Machine 2**
- (4.512 - 4.100) + (5.000 - 2.500) = 0.412 + 2.500 = 2.912
- Average: 2.912 / 2 = **1.456**

---

## 🔧 How to Solve

1. Pair each `start` activity with its corresponding `end` using a self-join.
2. Calculate the difference in timestamps to get processing time.
3. Group the results by machine and take the average.
4. Round the result to 3 decimal places.

---

## 🧾 SQL Query

```sql
WITH processtimes AS (
  SELECT 
    a.machine_id, 
    a.process_id, 
    (b.timestamp - a.timestamp) AS processing_time
  FROM Activity a
  JOIN Activity b
    ON a.machine_id = b.machine_id
    AND a.process_id = b.process_id
    AND a.activity_type = 'start'
    AND b.activity_type = 'end'
)

SELECT 
  machine_id, 
  ROUND(AVG(processing_time), 3) AS processing_time
FROM processtimes
GROUP BY machine_id;
