## A great desktop application is powerful and, at the same time, simple.
#### What is does "power" really mean in terms of software? 
- An application might be considered powerful if it is jam-packed full of features, having a tremendous breadth of functionality in an attempt to be all things to all users. **BUT, An application is powerful when it enables its target users to realize their full potential efficiently. Thus, the ultimate measure of power is productivity, not the number of features.**
- An application is powerful when it has the right combination of these characteristics:
	- **Enabling.** The application satisfies the needs of its target users, enabling them to perform tasks that they couldn't otherwise do and achieve their goals effectively.
	- **Efficient.** The application enables users to perform tasks with a level of productivity and scale that wasn't possible before.
	- **Versatile.** The application enables users to perform a wide range of tasks effectively in a variety of circumstances.
	- **Direct.** The application feels like it is directly helping users achieve their goals, instead of getting in the way or requiring unnecessary steps. Features like shortcuts, keyboard access, and macros improve the sense of directness.
	- **Flexible.** The application allows users complete, fine-grained control over their work.
	- **Integrated.** The application is well integrated with Microsoft Windows, allowing it to share data with other applications.
	- **Advanced.** The application has extraordinary, innovative, state-of-the-art features that are not found in competing solutions.
#### What makes a user experience simple?
- **Simplicity is the reduction or elimination of an attribute of a design that target users are aware of and consider unessential.**
- In practice, simplicity is achieved by selecting the right feature set and presenting the features in the right way. This reduces the unessential, both real and perceived.

#### Simplicity vs Ease of use
- Simplicity, when correctly applied, results in ease of use. But simplicity and ease of use are not the same concepts. **Ease of use is achieved when users can perform a task successfully on their own without difficulty or confusion within a suitable amount of time.** There are many ways to achieve ease of use, and simplicity—the reduction of the unessential—is just one of them.
![[Pasted image 20240304082753.png|center|500]]
## Design principles
- **To obtain simplicity, always design for the probable, not the possible.**
- <u>The possible</u>: Design decisions based on what's possible lead to complex user interfaces like the Registry Editor, where the design assumes that all actions are equally possible and as a result require equal effort. Because anything is possible, user goals aren't considered in design decisions.

- <u>The probable</u>: Design decisions based on the probable lead to simplified, goal- and task-based solutions, where the likely scenarios receive focus and require minimal effort to perform.
```ad-note
**To obtain simplicity, focus on what is likely; reduce, hide, or remove what is unlikely; and eliminate what is impossible.**
```
## Design technique
- To obtain simplicity while maintaining power, choose the **right set of features**, locate the features **in the right places**, and **reduce the effort** to use them. This section gives some common techniques to achieve these goals.
### 1 - Choosing the right feature set
```ad-note
"Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away." —Antoine de Saint-Exupery
```
- The following design techniques give your users the features they need while achieving simplicity through actual reduction or removal:
	- **Determine the features your users need.** Understand your users' needs through goal, scenario, and task analysis. Determine a set of features that realizes these objectives.
	- **Remove unnecessary elements.** Remove elements that aren't likely to be used or have preferable alternatives.
	- **Remove unnecessary redundancy.** There might be several effective ways to perform a task. To achieve simplicity, make the hard decision and choose the best one for your target users instead of providing all of them and making the choice an option.
	- **Make it "just work" automatically.** The element is necessary, but any user interaction to get it to work is not because there is an acceptable default behavior or configuration. To achieve simplicity, make it work automatically and either hide it from the user completely or reduce its exposure significantly.
### 2 - Streamlining the presentation
```ad-note
"The ability to simplify means to eliminate the unnecessary so that the necessary may speak." —Hans Hofmann
```
- Use the following design techniques to preserve power, while achieving simplicity through the perception of reduction or removal:
	- **Combine what should be combined.** Put the essential features that support a task together so that a task can be performed in one place. The task's steps should have a unified, streamlined flow. Break down complex tasks into a set of easy, clear steps, so that "one" place might consist of several UI surfaces, such as a wizard.
	- **Separate what should be separated.** Not everything can be presented in one place, so always have clear, well-chosen boundaries. Make features that support core scenarios central and obvious, and hide optional functionality or make it peripheral. Separate individual tasks and provide links to related tasks. For example, tasks related to manipulating photos should be clearly separated from tasks related to managing collections of photos, but they should be readily accessible from each other.
	- **Eliminate what can be eliminated.** Take a printout of your design and highlight the elements used to perform the most important tasks. Even highlight the individual words in the UI text that communicate useful information. Now review what isn't highlighted and consider removing it from the design. If you remove the item, would anything bad happen? If not, remove it!
	- Consistency, configurability, and generalization are often desirable qualities, but they can lead to unnecessary complexity. Review your design for misguided efforts in consistency (such as having redundant text), generalization (such as having any number of time zones when two is sufficient), and configurability (such as options that users aren't likely to change), and eliminate what can be eliminated.
	- **Put the elements in the right place.** Within a window, an element's location should follow its utility. Essential controls, instructions, and explanations should all be in context in logical order. If more options are needed, expose them in context by clicking a chevron or similar mechanism; if more information is needed, display an infotip on mouse hover. Place less important tasks, options, and Help information outside the main flow in a separate window or page. The technique of displaying additional detail as needed is called progressive disclosure.
	- **Use meaningful high-level combinations.** It is often simpler and more scalable to select and manipulate groups of related elements than individual elements. Examples of high-level combinations include folders, themes, styles, and user groups. Such combinations often map to a user goal or intention that isn't apparent from the individual elements. For example, the intention behind the High Contrast Black color scheme is far more apparent than that of a black window background.
	- **Select the right controls.** Design elements are embodied by the controls you use to represent them, so selecting the right control is crucial to efficient presentation. For example, the font selection box used by Microsoft Word shows both a preview of the font as well as the most recently used fonts. Similarly, the way Word shows potential spelling and grammar errors in place is much simpler than the dialog box alternative, as shown in the beginning of this article.
### 3 - Reducing effort
```ad-note
"Simple things should be simple. Complex things should be possible."—Alan Kay
```
- The following design techniques result in reduced effort for users:
	- **Make tasks discoverable and visible.** All tasks, but especially frequent tasks, should be readily discoverable within the user interface. The steps required to perform tasks should be visible and should not rely on memorization.
	- **Present tasks in the user's domain.** Complex software requires users to map their problems to the technology. Simple software does that mapping for them by presenting what is natural. For example, a red-eye reduction feature maps directly to the problem space and doesn't require users to think in terms of details like hues and gradients.
	- **Put domain knowledge into the program.** Users shouldn't be required to access external information to use your application successfully. Domain knowledge can range from complex data and algorithms to simply making it clear what type of input is valid.
	- **Use text that users understand.** Well-crafted text is crucial to effective communication with users. Use concepts and terms familiar to your users. Fully explain what is being asked in plain language so that users can make intelligent, informed decisions.
	- **Use safe, secure, probable defaults.** If a setting has a value that applies to most users in most circumstances, and that setting is both safe and secure, use it as the default value. Make users specify values only when necessary.
	- **Use constraints.** If there are many ways to perform a task, but only some are correct, constrain the task to those correct ways. Users should not be allowed to make readily preventable mistakes.