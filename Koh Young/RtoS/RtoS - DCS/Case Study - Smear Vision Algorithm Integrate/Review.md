# Notes
- When install new GUI version, check for the shortcut in C:\\KohYoung\\NIS directory
- To force clean (delete) changes in git repo use: **git clean -fd**
- **The View on InspectionItem means a view per each vision algorithm of an InspectionItem.** An inspection item can be related to many algoritms, not one. For example: SolderPasteInspectionItem include HeightReconstruction, HeightReconstructionPostProcessing, PlaneDetection,... By examining UX design, you can determine what becomes an InspectionItem. In RtoS, each Item Card visible in the UI corresponds to a single Inspection Item class. For example, the Item Cards for “Smear” and “Shape” are as follows, which means that “Smear” and “Shape” should each be represented by an InspectionItem class. **The algorithm for “Smear” is “SmearDimension,” and for “Shape” is “SolderShape.” Therefore, the relationship between InspectionItem and VisionAlgorithm is 1:1.** In this case, you do not need a View and can set HasView() => False.
- Old format for InspectionItemResult => Remove on sight:
![[Pasted image 20240603104549.png|center|800]]
- Change precision point of value:
![[Pasted image 20240603112341.png|center|800]]
- Rule for setting enum value in RecipeCode:
![[Pasted image 20240603114406.png|center|300]]
- ![[Pasted image 20240603114549.png]]
- ![[Pasted image 20240603114743.png]]
- ![[Pasted image 20240612103350.png]]