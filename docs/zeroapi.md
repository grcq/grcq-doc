# ZeroAPI

> ## Getting Started

Firstly, you will need to add the JAR file to your dependencies and add it to your plugins folder on your server.
To add the JAR file, create a folder and name it whatever you want. I highly recommend to name it <code>libs</code>.
Then go to your pom.xml and add the dependency like this:

**Maven**
```xml
<dependency>
    <groupId>cf.grcq</groupId>
    <artifactId>ZeroAPI</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/libs/ZeroAPI-1.0.jar</systemPath>
</dependency>
```

**Gradle**
```gradle
dependencies {
    compile files("$System.env.JAVA_HOME/libs/ZeroAPI-1.0.jar)
}
```

When you have added the dependency, you may now reload your project.


> ## Creating Commands

If you want to use the Command Handler from the API. You have to create an instance for it in the main class.
There are multiple ways to create an instance, and I will show one way. Firstly, you'll need to create a variable for it. 
The Command Handler class is named CommandHandler you can find it easily. Then you need to initalize the instance by doing<code>this.commandHandler = new CommandHandler(this);</code>. Using categories and doing <code>CommandHandler#registerAll</code> is deprecated and will not work. I will try to fix that in the future.

Next step, you need a class for the command and do the following:
- Extend the class with the Command class.
- Implement all the methods from the Command class.
- Add <code>@CommandInfo</code> annotation to the class and set the name, and permission if you want to set permissions for it.
- If you have enabled <code>requireArgs</code> in the <code>@CommandInfo</code> annotation, you don't need the execute() method. Choose the execute method from what you have put in as executor (Default: <code>CommandSender.class</code>). You can choose between <code>Player.class</code> and <code>CommandSender.class</code>
- In the <code>helpMessage()</code> method, create an array list of the arguments you want to show in the usage message.

> ## Creating Sub Commands

To create a sub commands for a command, you will need to create a new class that extends the class SubCommand.
Now, do the following:
- Implement all the methods from the SubCommand class.
- Add <code>@SubCommandInfo</code> annotation to the class and set the name, and permission if you want to set permissions for it.
- If you want the sub commands to use arguments, put <code>requireArgs</code> to true in the annotation.
- Create tab completion for the argument(s), or if you don't need tab completion, just return an empty array list.
- Add the sub command to the main command's class and add it to the <code>subCommands()</code> method.


> ## Using the Database API

**MongoDB**

...


**MySQL**

...

> ## Using the GUI API

If you want to create a GUI, then you firstly need to create a class and extend it with the GUI class. Then implement the methods from the class. Fill inn the methods with size, title, and buttons.
After that, you probably want to add items to the GUI. If so, then you will need to do the following:
- Create a class extending the class <code>Button</code>.
- Implement all the methods from the extended class.
- Fill in the methods with your materials, amount, damage value, lore, and name.
- Go to your menu class and create a <code>Map\<Integer, Button></code> variablie in the <code>getButtons(Player)</code> method. Add the buttons in the slots you want in the HashMap and return the variable. Example:
```java
Map<Integer, Button> buttons = new HashMap<>();
buttons.put(0, new ExampleButton()); // 0 is the first slot of the GUI
return buttons;
```
- Then create a new instance and open the menu for the player in a command or an event. Example: <code>new ExampleGUI().openMenu(player)</code>