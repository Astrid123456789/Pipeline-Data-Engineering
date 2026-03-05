# ELT Pipeline with dbt — GitHub Repository Scoring

## Overview

This lab consists of building a complete ELT pipeline to analyze the activity of several GitHub repositories and produce a scoring system that allows comparing open-source projects.

The pipeline follows the Medallion architecture with three layers: Bronze (raw data), Silver (cleaned and typed data), and Gold (analytical models and business metrics).

The project uses Python, dbt and DuckDB to ingest, transform and analyze the data. The goal is to transform raw GitHub activity (commits, pull requests and issues) into analytical tables and then compute a final scoring table that ranks repositories according to several criteria such as popularity, activity, responsiveness and community engagement.

---

## Results and Analysis

To answer the questions of the lab, we query the final Gold table `scoring_repositories` produced by dbt.

### Best overall repository

To identify which repository has the best overall score, we order repositories by their activity score using the following command:

    dbt show --inline "select score_activity, repo_name from main_gold.scoring_repositories order by score_activity desc"

The result shows that **react** has the best overall score.

### Most recently active repository

To determine which repository is the most recently active, we analyze the activity score of each repository using the following command:

    dbt show --inline "select repo_name, score_responsiveness from gold.scoring_repositories order by score_responsiveness desc"

The result shows that **tensorflow** is the most recently active repository.

### Fastest repository to respond to PRs

To find which repository responds the fastest to pull requests, we analyze the responsiveness score with the following command:

    dbt show --inline "select repo_name, score_responsiveness from main_gold.scoring_repositories order by score_responsiveness desc"

The result shows that **flask** responds the fastest to pull requests.

### Repositories with strong community but low recent activity

To identify repositories that have strong community engagement but relatively little recent development activity, we compare the community score and the activity score using the following query:

    dbt show --inline "select repo_name, score_community, score_activity, (score_community - score_activity) as difference from main_gold.scoring_repositories order by difference desc"

This query highlights repositories where the difference between community engagement and recent activity is the largest.

---

## Incremental Strategy Reflection

### Can a commit be modified after the fact? What impact does this have on strategy choice?

A commit is generally immutable. Once it is created, its content does not change because it is identified by a unique hash (SHA). If the history is rewritten using a rebase or a force push, new commits with new SHAs are created instead of modifying the original one.

Because of this behavior, an incremental strategy for commits can simply append new rows by filtering on the commit date. Existing commits usually do not need to be updated.

### An issue can change state (open → closed). How do you handle this case?

An issue can change state during its lifecycle, for example from open to closed.

To handle this situation, the pipeline must allow updates to existing rows. One solution is to use a merge strategy based on the issue identifier so that the row is updated when the state changes. Another possible approach is to use dbt snapshots to keep the history of state changes over time.

### Should the Gold layer also be made incremental? What problems does this cause?

It is technically possible to make the Gold layer incremental, but it introduces several complications.

Gold tables usually contain aggregated metrics or scores computed from multiple sources. If historical data changes, for example when an issue is closed later, previously computed metrics may also change. This would require recalculating past data and maintaining complex update logic.

For this reason, the Gold layer is often rebuilt completely instead of being incremental.
