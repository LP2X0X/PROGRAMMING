# Object Lifetime Overview
 
```ccard
type: folder_brief_live
```
 
%% Begin Waypoint %%
- [[Building Finalizable and Disposable Types]]
- [[Compact Memory Feature of GC]]
- [[Determining if an Object is Live]]
- [[Example of implementing IDisposable]]
- **[[Features]]**
- [[Forcing a Garbage Collection]]
- [[Garbage Collection Types]]
- **[[Keywords]]**
- [[Marshal class]]
- [[Object Generations]]
- [[Order of Actions when Dispose is called]]
- **[[Questions]]**
	- [[Are both dispose and finalization wait for the garbage collector]]
	- [[Can you still access member of an object after it has been dispose]]
	- [[Why Compacting]]
	- [[Why is it perfectly safe to communicate with other managed objects within a Dispose() method]]
- [[Rules]]
- [[System.GC]]
- [[The Basics of Object Lifetime]]

%% End Waypoint %%