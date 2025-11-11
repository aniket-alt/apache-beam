# Apache Beam Data Engineering Exercise 🚀

Google Colab link: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1cHgkJf8O1z8r3SJxF2lIJvbdMyNsBDeP?usp=sharing)

Video Tutorial: https://drive.google.com/drive/folders/1CDo_KDz8co34pRkyPzsLxZTM3eM7t2Zy?usp=sharing

This project is a Google Colab notebook created for a data engineering exercise. It demonstrates the core features and concepts of the **Apache Beam** batch and stream processing model by building a complete data pipeline.

---
Name: Aniket Anil Naik

SJSU ID: 019107114

---
## 🎯 Project Scenario: Video Game Event Stream

This pipeline simulates the processing of a live, windowed stream of events from a fictional online game.

Raw log strings (simulating data from a source like Pub/Sub or Kafka) are ingested. The pipeline then parses, validates, filters, and partitions these events. Finally, it performs time-based aggregations to calculate player scores within 10-second "chapters" of the game.

---

## ✨ Features Demonstrated

This pipeline is a practical demonstration of the following Apache Beam concepts as required by the assignment:

* **Pipeline IO:** Reading from an in-memory Python list (`beam.Create`) to bootstrap the pipeline and writing to the console (`beam.Map(print)`) as a "sink."
* **`ParDo`:** Used for the core parsing and timestamping logic. A `DoFn` (`ParseLogFn`) processes each string, handles potential errors (like a `try/except` block), and `yield`s a structured dictionary.
* **`Map`:** Used for simple 1-to-1 data transformations, such as extracting `(key, value)` pairs (e.g., `(player_id, damage)`) to prepare for aggregation.
* **`Filter`:** Used to discard any logs that are not valid game events (e.g., removing logs that failed parsing or have an unknown `event_type`).
* **`Partition`:** Used to split a single `PCollection` of clean logs into *two* separate `PCollection`s: one for **PvP** (Player vs. Player) events and one for **PvM** (Player vs. Monster) events.
* **`Windowing`:** Applying `beam.WindowInto(FixedWindows(10))` to chop the "infinite" stream of data into finite, 10-second, non-overlapping windows based on event timestamps.
* **Aggregation:** Using `beam.CombinePerKey(sum)` to calculate the total damage dealt by each player *within each window*.
* **Composite Transform:** The `ParseAndFilterLogs` class bundles the `ParDo` and `Filter` steps into a single, reusable, and clean `PTransform`. This makes the main pipeline graph much more readable.

---

## 🌊 Pipeline Flow

Here is a high-level overview of the pipeline's logic:

```
(Start)
   |
[Create Raw Logs]
   |
[ParseAndFilterLogs (Composite Transform)]
   |
   +---[Clean Logs PCollection]---+
   |                             |
[Partition (PvP/PvM)]     [Map to (Player, Damage)]
   /         \                   |
[PvP Logs]  [PvM Logs]      [Apply 10-sec Window]
   |           |                   |
(Print)     (Print)           [Sum Damage per Player]
                                 |
                             (Print Scores)
```

---

## 🏃 How to Run

1.  **Open in Google Colab:** Click the "Open in Colab" badge at the top of this README.
2.  **Install Dependencies:** The first code cell installs `apache-beam`.
    ```python
    !pip install apache-beam
    ```
3.  **Run All Cells:** Simply run all cells from top to bottom (**Runtime > Run all**).
4.  **View Output:** The pipeline's results (PvP logs, PvM logs, and windowed player scores) will be printed at the end of the notebook.

---

## 📊 Example Output

You will see output similar to the following in the console. Notice how the scores are grouped by window (the first set of scores is for Window 1, and the second set is for Window 2).

```
--- Running Pipeline ---
PVP_LOG: {'event_type': 'pvp', 'player_id': 'player_1', 'damage': 15}
PVM_LOG: {'event_type': 'pvm', 'player_id': 'player_2', 'damage': 50}
PVP_LOG: {'event_type': 'pvp', 'player_id': 'player_3', 'damage': 75}
PVM_LOG: {'event_type': 'pvm', 'player_id': 'player_1', 'damage': 25}
SCORES_PER_WINDOW: ('player_1', 40)
SCORES_PER_WINDOW: ('player_2', 50)
SCORES_PER_WINDOW: ('player_3', 75)
PVM_LOG: {'event_type': 'pvm', 'player_id': 'player_3', 'damage': 10}
PVP_LOG: {'event_type': 'pvp', 'player_id': 'player_2', 'damage': 150}
PVP_LOG: {'event_type': 'pvp', 'player_id': 'player_1', 'damage': 30}
SCORES_PER_WINDOW: ('player_2', 150)
SCORES_PER_WINDOW: ('player_3', 10)
SCORES_PER_WINDOW: ('player_1', 30)
--- Pipeline Finished ---
```
