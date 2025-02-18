---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* {list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well}

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Logic.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

Similarly, how an undo operation goes through the `Model` component is shown below:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Model.png)

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* has a need to manage a significant number of contacts
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: manage contacts faster than a typical mouse/GUI driven app


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (good to but might not have) - `*`

| Priority | As a …​                                    | I want to …​                     | So that I can…​                                                        |
| -------- | ------------------------------------------ | ------------------------------ | ---------------------------------------------------------------------- |
| `* * *`  | new user | have a comprehensive document that details every possible feature. | learn how to use a particular feature |
| `* * *`  | user | add new patient, either with or without further information about the patient | |
| `* * *`  | user | delete patient | free storage resources |
| `* * *`  | user | be able to add the contact information of my patient into the database | I can get in touch with them when needed or under emergency situation |
| `* * *`  | user | be able to  delete the contact information for a particular patient when the information is outdated | I can keep in touch with my patient |
| `* * *`  | user | list all patients and their contacts | |
| `* * *`  | intermediate user | add appointment to free time slot | easily schedule an appointment and find free time slot for it |
| `* * *`  | intermediate user | delete appointment to free time slot | remove reservation if the patient is unable to attend |
| `* * *`  | intermediate user | list appointments | I can see all the appointments. |
| `* * *`  | user | add medication instructions for the patient | have a better understanding of the medication history of my patient |
| `* * *`  | user | list all information about specific patient | see the detail information of the patient |
| `* *`  | user | be able to modify/update the information of a particular patient | I can keep the most-updated information |
| `* *`  | user | be able to modify an appointment details of a patient | I can keep the most-updated appointment information |
| `* *`  | new user | option for undo most precent change | I can quickly fix my mistyped command |
| `* *`  | beginner | set the reminder for an appointment | remind me when Im busy with work |
| `* *`  | user | mark a reminder as done | better track the undo work |
| `* *`  | user | mark some of the patients as the special focus | better track the state of illness of patients who are in a very serious state |
| `* *`  | expert user | have short forms of existing commands | save time on typing the commands |
| `* *`  | new user | interactive guide with sample data | quickly understand app's capabilities |
| `* *`  | Doctor who is colaborating with other doctor | ability to leave comments or annotations on shared patient records | communicate specific insights or recommendations to my colleague |
| `* *`  | intermediate user | see the previous doage that the patient take in his medical record page | Adjust the dosage for the patient according to his/her state of illness |
| `* *`  | intermediate user | search about the patient's allergy history | Prescribing safe medication for patients |
| `* *`  | intermediate user | know about the previous illnesses of the patient | help to diagnose causes more accurately |
| `* *`  | intermediate user | be able to update the patient's condition upon the previous appointment | better track the sate of illness of the patient |
| `*`  | new user | suggestion on correction for mistyped commands | avoid typing wrong comands |
| `*`  | user | have a way to assign specific colors to specific tags | better differentiate the existing tags |
| `*`  | user | have a icon or button beside a feature that shows a tooltip when hovered |  quickly find out information about the feature without needing other references |
| `*`  | user | have a method of giving feedback to the developers | share aspects of the product that I would like changes to |
| `*`  | new user | have the ability to switch to a more simplified and beginner friendly UI | more effectively learn the basics |
| `*`  | trainee doctor | be able to refer to similar previous cases | study treatment that helps with this kind of cases |
| `*`  | experienced user | an efficient way to export and backup patient data | ensure the safety and accessibility of important information |
| `*`  | intermediate user | export selected patient's information | give the patient their personal information after they change their doctors, and delete the patient's information from the database safely |
| `*`  | intermediate user | export selected medical instructions | The patient can follow my medical instructions closely |
| `*`  | intermediate user | see the list of patients based on the next follow-up meeting/calling time | have a better plan for my time and know which patient needs my attention next |

### Use cases

(For all use cases below, the **System** is the `VitalConnect` and the **Actor** is the `user`, unless specified otherwise)

**Use case: UC1 - Add a patient**
**MSS**
1.  User requests to add a patient by keying the patient's name and NRIC in the command.
2.  VitalConnect adds the patient's name and NRIC.
Use case ends.
**Extensions**
* 1a. The NRIC already exists in the system.
      * 1a1. VitalConnect displays warning message and the existing patient's information.
      Use case ends.
* 1b. The NRIC or name entered is invalid.
      * 1b1. VitalConnect shows an error message.
      Use case ends.

**Use case: UC2 - Delete a patient**
**MSS**
1.  User requests to delete a patient by keying the patient's name or NRIC in the command.
2.  VitalConnect deletes the patient from database.
Use case ends.
**Extensions**
* 1a. The patient doesn't exist in the system.
      * 1a1. VitalConnect displays an error message.
      Use case ends.

**Use case: UC3 - Add an appointment**
**MSS**
1.  User requests to add an appointment for a patient.
2.  VitalConnect add the appointment to the database under this patient's NRIC.
Use case ends.
**Extensions**
* 1a. Critical information (time and doctor) missing in the appointment.
      * 1a1. VitalConnect displays a warning message.
      Use case ends.
* 1b. The assigned patient doesn't exist in the database.
      * 1b1. VitalConnect displays a warning message.
      Use case ends.
* 1c. The appointment time crashes with existing time.
      * 1c1. VitalConnect displays a warning message.
      * 1c1. VitalConnect displays the appointment with crashing time.
      Use case ends.

**Use case: UC4 - Delete an appointment**
**MSS**
1.  User requests to delete an appointment for a patient.
2.  VitalConnect removes the appointment from the database.
Use case ends.
**Extensions**
* 1a. The assigned patient or the appointment doesn't exist in the database.
      * 1a1. VitalConnect displays a warning message.
      Use case ends.

**Use case: UC5 - Modify an appointment**
**MSS**
1.  User requests to modify an appointment for a patient by keying the appointment's id.
2.  VitalConnect displays the detail of the appointment to be modified.
3.  User specify which field to be modified and enters the new information.
4.  VitalConnect saves the new appointment information.
5.  VitalConnect displays the updated detail of the appointment modified.
Use case ends.
**Extensions**
* 1a. The appointment refered by the id doesn't exist in the database.
      * 1a1. VitalConnect displays an error message.
      Use case ends.
* 1b. The id is not a valid number.
      * 1b1. VitalConnect displays an error message.
      Use case ends.
* 3a. The field to be modified is unrecognized.
      * 3a1. VitalConnect displays an error message.
      * 3a2. VitalConnect request for valid field information.
      * 3a3. User enters new field information.
      Steps 3a1-3a3 are repeated until the data entered are correct.
      Use case resumes from step 4.
* 3b. The new information is in invalid form or contains invalid character.
      * 3b1. VitalConnect displays an error message.
      * 3b2. VitalConnect request for valid data entry.
      * 3b3. User enters new field information.
      Steps 3b1-3b3 are repeated until the data entered are valid.
      Use case resumes from step 4.
* 3c. The appointment time crashes with existing time.
      * 3c1. VitalConnect displays an error message.
      * 3c2. VitalConnect displays the appointment with crashing time.
      * 3c3. VitalConnect request for valid data entry.
      * 3c4. User enters new field information.
      Steps 3c1-3c4 are repeated until the time doesn't crash.
      Use case resumes from step 4.

**Use case: UC6 - Add contact information**
**MSS**
1.  User requests to add contact information for a patient.
2.  VitalConnect save the contact information to the database.
Use case ends.
**Extensions**
* 1a. The patient doesn't exist in the database.
      * 1a1. VitalConnect displays a warning message.
      Use case ends.
* 1b. The contact information is invalid.
      * 1b1. VitalConnect displays a warning message.
      Use case ends.

**Use case: UC7 - Delete contact information**
**MSS**
1.  User requests to delete contact information for a patient.
2.  VitalConnect remove the contact information to the database.
Use case ends.
**Extensions**
* 1a. The patient or contact information doesn't exist in the database.
      * 1a1. VitalConnect displays a warning message.
      Use case ends.

**Use case: UC8 - Modify contact information**
**MSS**
1.  User requests to modify contact information for a patient.
2.  VitalConnect displays the updated contact information of the patient.
Use case ends.
**Extensions**
* 1a. The patient or contact information doesn't exist in the database.
      * 1a1. VitalConnect displays a warning message.
      Use case ends.
* 1b. The contact information is invalid.
      * 1b1. VitalConnect displays a warning message.
      Use case ends.

### Non-Functional Requirements

# Technical Requirements
1. Should work on any _mainstream OS_ as long as it has Java `11` or above installed.

# Performance Requirements
1. Should be able to hold up to 100 patients with 1000 appointments without a noticeable sluggishness in performance for typical usage.
2. The system should respond within 3 seconds.

# Quality Requirements
1. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
2. System should be robust for any form of data file crashes and invalid user input.

# Scope
1. The product will <strong>NOT</strong> enforce any form of protection of the generated data file containing patients' information. The organization should be responsible for ensuring the safety of their patient's data.

# Process Requirements
1. The project is expected to grow in breadth-first iterative process.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **CLI**: Acronym for Command Line Interface, a text-based interface where users interact with the application by typing commands.
* **VitalConnect**: The system being described, representing the medical management application.
* **User**: The person interacting with the VitalConnect system to perform various actions.
* **Use Case**: A specific scenario or situation in which a user interacts with the VitalConnect system to achieve a specific goal.
* **MSS (Main Success Scenario)**: The primary sequence of steps in a use case that represents the successful accomplishment of the user's goal.
* **Extensions**: Additional scenarios that may occur during the execution of a use case, usually describing alternative paths or error-handling situations.
* **NRIC**: National Registration Identity Card, a unique identification number used in some countries.
* **Database**: A structured set of data stored electronically, in this context, referring to the storage system for patient and appointment information.
* **Appointment**: A scheduled meeting or arrangement, often referring to a scheduled medical consultation in the context of VitalConnect.
* **Field**: In the context of modifying an appointment, a specific piece of information within the appointment details that the user can choose to modify (e.g., time, doctor).
* **ID (Identification Number)**: A unique identifier associated with a specific appointment, used to distinguish and reference individual appointments.
* **Contact Information**: Information related to how the system can reach or contact a patient, such as phone number or email address.
* **Warning Message**: An alert displayed by the VitalConnect system to notify the user of a potential issue or discrepancy.
* **Error Message**: A notification displayed by the VitalConnect system to inform the user about a critical issue or mistake.
* **Crashing Time**: A situation where the proposed time for an appointment conflicts with an existing appointment time in the system.
* **Invalid Data Entry**: Information entered by the user that does not meet the required format or criteria.
* **Valid Data Entry**: Information entered by the user that meets the required format or criteria.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
