---
title: "Pull mirroring"
menuTitle: Pull mirroring
force_searchable: true
description: Configuring pull mirroring in a repository
permalink: en/code/documentation/user/pull-mirroring.html
weight: 50
---

To configure repository mirroring, on the project page, go to "Settings" → "Repository" → "Mirroring repositories".

Mirroring pulls data into an existing repository and does not create one. For an empty repository, there are two options:

- When importing by URL (the "Import project" → "Repository by URL" form, both when creating a new project and when importing into an existing one), select the "Mirror repository" checkbox: the repository is imported and set up for pull mirroring in one step.
- Alternatively, import the repository first, and then configure mirroring on the "Settings" → "Repository" → "Mirroring repositories" page.

The "Mirror repository" checkbox is active only if repository mirroring is allowed in the instance settings. After the initial import, subsequent synchronization runs automatically on schedule.

All hooks are triggered during mirroring, and pulling a large repository may significantly impact system performance.

## Configuring pull mirroring of a repository

{% alert level="warning" %}
Only one pull mirroring task can be configured per project. Multiple push mirroring tasks are allowed.
{% endalert %}

To configure pull mirroring of a repository, follow these steps:

1. Go to the project page:

   - Open your project in the Deckhouse Code interface.  
   - In the left-hand menu, select "Settings" → "Repository".  
   - Scroll down to the "Mirroring repositories" section.

1. Specify the repository URL.
   Credentials in the URL are ignored.
   Use the fields in the "Authentication method" block below for authorization.

1. Configure authentication:
   - If using HTTP(S) access, in the "Authentication method" field, select "Username and password" and enter:
     - "Username": Your username.
     - "Password": Your password or access token.
   - If using SSH mirroring, specify the username (typically `git`). After saving the configuration, Deckhouse Code will generate an SSH key to be used for access.

## Scheduling and error handling

- Pull mirroring tasks are scheduled once per hour (`Projects::PullMirrorScheduleWorker`).  
- Each mirror is updated no more than once every 6 hours.  
- If a mirroring task fails, the next attempt will be during the next run of `Projects::PullMirrorScheduleWorker`.
  The maximum number of retry attempts is **5**.
  Clicking "Update now" resets the failure counter.
- The initial import of a mirror created via the "Mirror repository" checkbox is retried on the same schedule when it fails and consumes the same retry limit. A successful import resets the counter.
- If a Sidekiq task terminates unexpectedly (for example, due to a Sidekiq restart), its status is updated after 3 hours, and a new synchronization attempt is scheduled.

## LFS (Large File Storage) specifics

When using pull mirroring, LFS objects are fetched **only if** LFS is enabled in the target Deckhouse Code project.
