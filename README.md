# JabRef
* website: <https://www.jabref.org/>
* GitHub Repository: <https://github.com/JabRef/jabref>
* Contributors Guide: <https://github.com/JabRef/jabref/blob/master/CONTRIBUTING.md>

### Description

Following the [Contributing Guide](https://github.com/JabRef/jabref/blob/master/CONTRIBUTING.md) provided by the JabRef team, we plan to start by [Setting up a local workspace](https://github.com/JabRef/jabref/wiki/Guidelines-for-setting-up-a-local-workspace) under the Eclipse IDE. Once we installed the IDE well Fork JabRef into your GitHub account, Clone our forked repository on your local machine, generate additional source codes and getting dependencies using Gradle for finally building it and getting it running as a Java Application. We plan on using the [guide to pull request]( https://github.com/blog/1943-how-to-write-the-perfect-pull-request) for requesting the developers a branch pull as specified in the guide. Then we�ll start by reading the [Coding Guide]( https://github.com/JabRef/jabref/wiki/Code-Howtos) and adding a CHANGELOG.md file as specified in the guide. Once that is done, we will start by fixing issues registered on the [beginners section]( https://github.com/JabRef/jabref/labels/beginner) to get a flavor of the code, for finally working in issues with [more impact]( https://github.com/JabRef/jabref/labels/help-wanted) while following the [FAQ Guide]( http://help.jabref.org/en/FAQcontributing).

## Summary

We will be working on the JabRef project, which is an open source bibliography reference manager. In short, this application gives the user the ability to add, search, and sort all their references in one place. We plan to contribute to the project by first following the contributing guide on creating a formal pull request and setting up our local workspace using the Eclipse IDE. Once that is done we will need to edit their CHANGELOG.md file in order to accurately document the changes we intend to make. After this is setup and we have created a fork to begin working on the project, we will turn our attention to the beginners section to become familiar with how the code works�this is also the same section that will allow us to contribute to their code more quickly because we will be working on existing bugs in the software. From there, if time permits, we can begin working on issues that have more impact.

## Project Plan Steps  

### 1. Setting-Up the Development Environment  
Our first step was to fork the repository from the JabRef team. Once this was complete, we decided that it would be easier for us to work as a team on our spell checking feature if one of us added the other as a contributor to their forked repository from JabRef. This workflow method allowed us to contribute to the same feature and made it easier to manage the changes we each worked on. In addition, and if the JabRef team accepts our changes, it will make it easier to pull those changes back into their code.  
  
The JabRef team provided some tips for setting up our local workspace, which can be found here: <https://github.com/JabRef/jabref/wiki/Guidelines-for-setting-up-a-local-workspace>. We decided to use Eclipse for this project and followed their guidelines for setting up our workspace and used Gradle to build the project. Once the workspace was setup, we were ready to begin going through the code to gain a better understanding of how it worked.  
### 2. Understanding The Application  
With this being the first open source project that each of us has ever contributed to, it was a great challenge and learning experience to not only use Github's fork and branch workflow correctly but to also understand someone else's code, especially one as large and complex as JabRef. After importing and setting up the project in our IDE, the next few days were dedicated to just going through the code to gain a deeper understanding and familiarity with how the application was working. Before we began working on the code in our IDE, we each downloaded their standalone application to gain a better understanding from the user standpoint of how JabRef is supposed to work. It was during these "user testing" sessions that we came up with an idea for the feature we wanted to implement--and also after reviewing the issue tracker JabRef setup on their Github page and realizing there was no one working on the idea we had in mind. 
### 3. Identifying a desired contribution  
Once we figured out what feature we wanted to add to JabRef, we thought it would be best to look for a spell checker API that we could plug into the application and begin to configure. The spell checking API that as used for this project is called Jazzy, a link to the website can be found here: [Jazzy](http://jazzy.sourceforge.net/). We decided that as a first version of the spell checker system we would create it to only be used in the "Required Fields" section of the application--this is where the user enters information about the source they're using, such as book title or author name. See this image as an example: 
![Imgur](http://i.imgur.com/NT4ggTu.png)                                               
### 4. Examining the code  
![Most Important Classes and their Relation](https://yuml.me/20975ef4)  
We started by reading the [High Level Documentation](https://github.com/JabRef/jabref/wiki/High-Level-Documentation) and paying attention to two main things: package structure and the most important classes and their relation.
After doing so we followed the path by debugging, from the class containing the main method to two of the classes that where involved in the new feature we wanted to create:
![JafRef Classes](http://i.imgur.com/0cnK66S.jpg)
* **JafRefFrame**  
Contains all the components being displayed while the application runs.
* **EntryEditor**  
Container that extends from JPanel and implements EntryContainer, this allows it to create the controls being used over the toolbar and the possible action over the entries
* **SpellCheckerAction**  
Inner class of EntryEditor that provides the functionality for the spell checking tool over the different fields of the entry.  

### 5. Implementing our contribution
![Class Diagram](http://i.imgur.com/RW3E27Z.png)
#### A. Define Structure
We had to adapt our design to the way JabRef works and came up with the following classes:  
* **SpellCheckAbstract:** Interface that allows alternative implementations for the spell check engine. 
* **JazzySpellChecker:** Contains the needed methods to use the Jazzy libraries inside of JabRef and perform the spell check on the entries.  
* **SpellCheckerDialog:** Dialog that helps performing the corrections task on the GUI of JabRef for each of the present Fields of the current entries.  
* **SpellCheckerAction:** A class which extends from AbstracAction and help us add the needed option on the toolbar of the entries.  
* **SpellCheckerRecord:** Simple POJO (Plain Old Java Object) that helps us store one set of words to be corrected or ignored by the SpellCheckerDialog.  
#### B. Code Reuse
In the following example we are creating our own class of type AbstracAction. This way we can reuse all the code written behind every present button on the toolbar of each of the entries.
```
    private class SpellCheckerAction extends AbstractAction 
    {
        JazzySpellChecker jazzySpellChecker = null;
        SpellCheckerDialog dialog;
        private SpellCheckerAction() 
        {
            super(Localization.lang("Generate BibTeX key"), IconTheme.JabRefIcon.MAKE_KEY.getIcon());
            putValue(Action.SHORT_DESCRIPTION, Localization.lang("Generate BibTeX key"));
            jazzySpellChecker = new JazzySpellChecker();
        }
        @Override
        public void actionPerformed(ActionEvent e) 
        {
			System.out.println(panel.getCurrentEditor().getEntry().getFieldMap());
            // 1. Perform a Spell Check on the fields that contains any kind of text
            List<SpellCheckerRecord> allErrors = jazzySpellChecker.performSpellCheck(panel.getCurrentEditor().getEntry().getFieldMap());
            dialog = new SpellCheckerDialog(frame, panel, entry, allErrors);
            dialog.setVisible(true);
        }
    }
```
#### C. Test Cases
* **SpellCheckerDialogTest**  
This JUnit Test Class takes care of testing the: Ignore, Replace, Cancel and Select options from the SpellCheckerDialog class.
* **EntryEditorSpellCheckerTest**  
This JUnit Test Class takes care of testing the Jazzy correct initialization and Its correct functionality inside of the EntryEditorSpellChecker class.
#### D. Spell Checker In Action
![Spell Checker](http://i.imgur.com/RSYsh3q.png)

As an example, you can see here we wanted to add the 1987 book by Stephen King titled Misery to our references manager. When we misspelled "mysery" the spell checker suggestion box identified that and suggested "misery" instead.

### 7. Applying Required Project Standards

Some of the aspects JabRef developers were suggesting at the [Code's how to](https://github.com/JabRef/jabref/wiki/Code-Howtos) requires us to:  
* Try not to abbreviate names of variables, classes or methods
* Use lowerCamelCase instead of snake_case
```
    private static final long serialVersionUID = 1L;
    private final JPanel contentPanel = new JPanel();
    JList<String> list;
    JLabel misspelledWord;
    JLabel localCurrentField;
    List<SpellCheckerRecord> wordsToCheck;
    int currentWord;
    BasePanel panel;
    BibEntry entry;
    JLabel lblNewLabel;
```

#### A. Throwing and Catching Exceptions
- All Exceptions we throw should be or extend JabRefException; This is especially important if the message stored in the Exception should be shown to the user. JabRefException has already implemented the getLocalizedMessage() method which should be used for such cases (see details below!).  
- Catch and wrap all API exceptions (such as IOExceptions) and rethrow them.  
Example:  
```  
try {
    // ...
} catch (IOException ioe) {
    throw new JabRefException("Something went wrong...", 
         Localization.lang("Something went wrong...", ioe);
}
```  
- Never, ever throw and catch Exception or Throwable.  
- Errors should only be logged when they are finally caught (i.e., logged only once). See Logging for details.  
- If the Exception message is intended to be shown to the User in the UI (see below) provide also a localizedMessage (see JabRefException).  
```  
    public JazzySpellChecker() {
        try {
            // Buffer the dictionary
            BufferedReader reader = new BufferedReader(new InputStreamReader(getClass().getResourceAsStream("/english.0")));
            // Load values for Dictionary from the internal Jazzy Jar
            dictionary = new SpellDictionaryHashMap(reader);
            // Create spell check object
            spellChecker = new SpellChecker(dictionary);
        } catch (IOException ioe) {
            //Verify we are able to find the file of the dictionary
            LOGGER.warn(ioe.getMessage(), ioe);
        }
    }
```  

#### B. When adding a library
Please try to use a version available at jCenter and add it to build.gradle. In any case, describe the library at external-libraries.txt. We need that information for our package maintainers (e.g., those of the debian package). Also add a txt file stating the license in libraries.
```  
Id:      com.swabunga.spell
Project: Jazzy
URL:     https://sourceforge.net/projects/jazzy/
License: LGPL 2.0
```  
```
compile 'com.swabunga.spell:0.5.3
```  
#### B. When adding a new Localization.lang entry*
Add new Localization.lang("KEY") to Java file. Tests fail. In the test output a snippet is generated which must be added to the English translation file. There is also a snippet generated for the non-English files, but this is irrelevant. Add snippet to English translation file located at src/main/resources/l10n/JabRef_en.properties With gradlew localizationUpdate the "KEY" is added to the other translation files as well. Tests are green again.
```  
public SpellCheckerDialog(JFrame frame, BasePanel panel, BibEntry entry, List<SpellCheckerRecord> wordsToCheck) {
        // This could also be replaced for a JabRefDialog type of object, this creates problems at the moment of performing test cases
        super(frame);
        this.wordsToCheck = wordsToCheck;
        this.entry = entry;
        this.panel = panel;
        // Code for creating the Dialog
        setTitle(Localization.lang("Entry_Fields_Spell_Check"));
        setBounds(100, 100, 365, 262);
        getContentPane().setLayout(new BorderLayout());
        contentPanel.setBorder(new EmptyBorder(5, 5, 5, 5));
        getContentPane().add(contentPanel, BorderLayout.CENTER);
        contentPanel.setLayout(null);
        setDefaultCloseOperation(JDialog.DISPOSE_ON_CLOSE);
        JPanel buttonPane = new JPanel();
        buttonPane.setLayout(new FlowLayout(FlowLayout.RIGHT));
        getContentPane().add(buttonPane, BorderLayout.SOUTH);
        // Label displaying the current field title
        lblNewLabel = new JLabel(Localization.lang("Current_Field_Spell_Check"));
        lblNewLabel.setFont(new Font("Tahoma", Font.BOLD, 13));
        lblNewLabel.setBounds(24, 81, 120, 16);
        contentPanel.add(lblNewLabel);
```  
```  
Spell_Check=Spell Check
Entry_Fields_Spell_Check=Entry Fields Spell Checker
Current_Field_Spell_Check=Current Field
Misspelled_Word_Spell_Check=Misspelled Word
Ignore_Spell_Check_Word=Ignore
Replace_Spell_Check_Word=Replace
Cancel_Spell_Check_Word=Cancel
No_Errors_Spell_Check_Word=No Errors Found!
```  
#### C. Test Cases  
![Tests are green](http://i.imgur.com/j600qQM.png)  
We focused on testing the two main classes we created for this functionality: SpellCheckerDialog and the EntryEditorSpellChecker. We have about 8 cases that try to discover bugs in our code and guarantee a good functionality.
```  
    @Before
    public void setUp() throws Exception {
        reader = new BufferedReader(new InputStreamReader(getClass().getResourceAsStream("/english.0")));
        dictionary = new SpellDictionaryHashMap(reader);
        spellChecker = new SpellChecker(dictionary);
        jazzySpellChecker = new JazzySpellChecker();
        currentFields = new LinkedHashMap<>();
        currentFields.put("author", "teh");
    }

    @Test
    public void getSuggestionsReturnsCorrectListOfWords() {
        String[] expectedValues = {"tea", "the", "ten"};
        List<Word> vectorSuggestedValues = spellChecker.getSuggestions("teh", 1);
        String[] suggestedValues = new String[vectorSuggestedValues.size()];
        for (int i = 0; i < vectorSuggestedValues.size(); i++) {
            suggestedValues[i] = vectorSuggestedValues.get(i).getWord();
        }
        //Verifying size of suggestions
        Assert.assertEquals(expectedValues.length, suggestedValues.length);
        //Verifying content of suggestions
        Assert.assertArrayEquals(expectedValues, suggestedValues);
    }
```  
#### D. Adding Comments  
As you can see, we added comments to the class, methods and each of the lines inside of the methods.
```  
import com.swabunga.spell.engine.Word;
import com.swabunga.spell.event.SpellChecker;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

/**
 * Helps us implementing and using the Jazzy methods inside of the fields of the current Entry as we require
 * Mainly focus on testing and implementing possible problems with the Jazzy Library
 * Jazzy could also be replaced in this class by a different Spell Check Engine
 */
public class JazzySpellChecker implements SpellCheckAbstract {

    private static final Log LOGGER = LogFactory.getLog(PreferencesDialog.class);
    SpellDictionaryHashMap dictionary = null;
    SpellChecker spellChecker = null;

    public JazzySpellChecker() {
        try {
            // Buffer the dictionary
            BufferedReader reader = new BufferedReader(new InputStreamReader(getClass().getResourceAsStream("/english.0")));
            // Load values for Dictionary from the internal Jazzy Jar
            dictionary = new SpellDictionaryHashMap(reader);
            // Create spell check object
            spellChecker = new SpellChecker(dictionary);
        } catch (IOException ioe) {
            //Verify we are able to find the file of the dictionary
            LOGGER.warn(ioe.getMessage(), ioe);
        }
    }

    /**
     * Method to perform the spell check over current fields of the Entry
     * @param currentFields: map containing information from the Entry, the id of the field and the text written by the user on the entry
     * @return List<SpellCheckerRecord> : LinkedList containing the words and the suggestions given by Jazzy
     */

    @Override
    public List<SpellCheckerRecord> performSpellCheck(Map<String, String> currentFields) {
        List<SpellCheckerRecord> allErrors = new ArrayList<>();
        // Verify the fields have content
        if (!currentFields.isEmpty()) {
            // Iterate over current fields
            for (Map.Entry<String, String> currentField : currentFields.entrySet()) {
                // Divide the String into words
                String[] arrayOfWords = currentField.getValue().split(" ");
                if (arrayOfWords.length > 0) {
                    // Iterate over the array of words we just created
                    for (int i = 0; i < arrayOfWords.length; i++) {
                        // Obtain the list of suggested replacements by the Jazzy lib (word, threshold)
                        List<Word> possibleWords = spellChecker.getSuggestions(arrayOfWords[i], 1);
                        // Verify the word is not present into the suggestions
                        if ((possibleWords != null) && (possibleWords.size() > 0) && !wordIsInSuggestion(possibleWords, arrayOfWords[i])) {
                            // Add word to a list to be processed as we require later on
                            allErrors.add(new SpellCheckerRecord(currentField.getKey(), arrayOfWords[i], i, possibleWords));
                        }
                    }
                }
            }

        }
        return allErrors;
    }

}
```  
### 8. Creating a Pull Request  
* **Title**  
JabRef Spell Checker  
* **Head**  
We need to know the name of the branch where our changes are implemented  
* **Base**  
We need to know the name of the branch we want our changes pulled into  
* **Body**  
My teammate and I are both graduate software engineering students at Loyola University Chicago our names are Tyree and Percy. We were enrolled in an open source computing class and decided to work with the JabRef repository as our final project for the course. Both of us thought it was an intersting project to begin learning about how to contribute to open source software and documentation. The purpose of this pull request is to add a spell checking feature to the required text fields in JabRef. With this spell checking feature implemented, users will have the ability to ignore a word suggestion, replace a word suggestion, or cancel the spell checking feature entirely. We thought this could be a useful feature to add since sometimes people may make mistakes in any number of the required text fields for a given reference, and correct spelling becomes especially important when the user needs to search many entries in their reference list.  
In order to use this feature the user needs to click on the spell checking icon located in the left toolbar under the "Change entry type" button. The button has a capital "A" with a check symbol directly underneath it, see example below:  
![Imgur](http://i.imgur.com/YgtSogz.png)  
Once the user has entered information in any of the text fields, they can press the spell check icon and it will begin looking through all the text fields and make suggestions for any words that might be misspelled.  See below for an example:  
For this example, I decided to input the novel by Stephen King titled Misery. In the first screenshot you can see that the spell checker is looking at the author field first and giving some spelling suggestions based on what was entered.  The spell checker first checks the first name of the author and then moves onto the author's last name (indicated in the second image) before completing the spell check of this field. 
![Imgur](http://i.imgur.com/mDNjBre.png)  
![Imgur](http://i.imgur.com/D6za2kI.png)  
For the sake of this example I skipped over a few fields to show how it handles a misspelled word. As you can see from the last image, the word "Misery" is spelled incorrectly in the text field with the spelling of "Mysery." The spell checker correctly identifies "Mysery" as a misspelled word and suggests the correct spelling of the word.  
![Imgur](http://i.imgur.com/4rmREhE.png)  
Thank you very much for taking the time to review and consider our pull request. My teammate and I learned a lot about working with the open source community through the JabRef project. Any feedback or suggestions about our code or the new spell checking feature would be welcomed and greatly appreciated. For more information about the changes we made to the project you can visit this webpage:  
[Spell Check for JabRef](https://github.com/psolizrodriguez/Open_Source_Project)  

### 9. Recording a Screen Cast

[![JabRef Spell Check Contribution Report](http://i.imgur.com/ZJc4JT6.png)](https://www.youtube.com/watch?v=H0sv3_mYFho)

