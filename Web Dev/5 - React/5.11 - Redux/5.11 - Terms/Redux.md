---
tags: 
 - react
 - term 
 - redux
 - state
 - management
---

![[Pasted image 20251114081525.png|center|700]]
![[Pasted image 20251114093235.png|center|700]]
![[Pasted image 20251114093633.png|center|700]]

Redux is not used as much lately because most apps today don’t actually have a lot of **global UI state**—the kind of state Redux was originally designed to manage. Modern React apps increasingly rely on **remote state** (server data) rather than large amounts of client-side global state. And remote state is better handled by tools like **React Query, SWR, Apollo, tRPC, RTK Query**, etc., which provide caching, background refetching, deduplication, stale-while-revalidate, and synchronization with servers—features Redux was never meant to solve. As a result, many teams only keep small amounts of UI state locally with React, while letting server-state libraries handle everything else. This shift reduces the need for Redux in many applications.