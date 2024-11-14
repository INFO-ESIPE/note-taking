[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## Observer

From the Gang of Four once again, this pattern **allows an object to be notified of another object's state changes** without creating strong coupling between the two classes.  

Two key ideas:
1. There is a common interface for all **observers**
2. An **Observable** allows **observers** to **register**, and will **notify them** in case of any change 
> E.g., Execute an action for each commit on a git repository, re-draw a window when the monopoly board changes...

![[observer.png]]


****
## Example: Game

We start with the following code:
```java
public class Game{
    private String homeTeam;
    private int scoreHomeTeam;
    private String awayTeam;
    private int scoreAwayTeam;

    public String getHomeTeam() { ... }

    public String getAwayTeam() { ... }

    public int getHomeTeamScore() { ... }

    public int getAwayTeamScore() { ... }

    public void startGame(){ ... }

    public void endGame() { ... }

    public void setScore(int scoreHomeTeam, int scoreVisitorTeam) { ... }
}
```

We want to:
- Send a tweet when there is a goal
- Post an article displaying the final score when the match is over
- Summarise the game in a PDF file


We set up a common interface (1) where all observed actions are:
```java
public interface GameObserver{
    public void onGameStart(Game game); // For PDF summary 
    public void onGameStop(Game game); // For article and PDF summary
    public void onScoreChange(Game game); // For Tweet and PDF summary
}
```

And we change our game so it becomes observable:
```java
public class Game {
    private String homeTeam;
    private int scoreHomeTeam;
    private String awayTeam;
    private int scoreAwayTeam;
    /* The list of observers we must notify */
    private final List<GameObserver> observers = new ArrayList<>();

	/* Add a new observer */
    public void register(GameObserver obs){
        observers.add(Objects.requireNonNull(obs));
    }

	/* Optional but still good feature to have */
    public void unregister(GameObserver obs){
        if (!observers.remove(Objects.requireNonNull(obs))) { 
	        throw new IllegalStateException(); 
	    }
    }

    public void startGame(){ 
       notifyGameStart();
       // ...
    }

    private void notifyGameStart(){
        for(var obs : observers){  // notify all observers
            obs.onGameStart(this);
        }
    }
    // ...
}
```


Now, let's begin by defining the **Twitter observer**:
```java
public class TwitterObserver implements GameObserver {
    @Override
    public void onGameStart(Game game){
        // nothing, we are not interested by that
    }
    
    @Override    
    public void onGameStop(Game game){
        // nothing, we are not interested by that
    }
    
    @Override    
    public void onScoreChange(Game game){
        TwitterAPI.tweet(game.getHomeTeam() + ": " + game.getHomeTeamScore() + " versus " 
                       + game.getAwayTeam() + ": " + game.getAwayTeamScore());
    }
}
```


As we see, there might be actions we aren't interested in for a specific observer. As the observed actions might grow on with time, it seems like a better choice to specify **default functions in the Interface** instead of abstract ones:
```java
public interface GameObserver {
    public default void onGameStart(Game game) { }
    public default void onGameStop(Game game) { }
    public default void onScoreChange(Game game) { }
}

/* 
This way, we only override the functions we are interested in, and don't end up with several empty functions we don't care about... 
*/
public class TwitterObserver implements GameObserver {
    @Override
    public void onScoreChange(Game game){
        TwitterAPI.tweet(game.getHomeTeam() + ": " + game.getHomeTeamScore() + " versus " 
                       + game.getAwayTeam() + ": " + game.getAwayTeamScore());
    }
}
```


Now, let's define the **Article Summary observer**.
Don't hesitate to use a classic internal state to keep track of some details that aren't stored/accessible from the observed class:
```java
public class SummaryObserver implements GameObserver {
    private int scoreChanges; // state

    @Override
    public void onGameStop(Game game){
        PublishAPI.post("After " + scoreChanges + " score changes, the game ends with "
                      + game.getHomeTeamScore() + " for " + game.getHomeTeam() + " versus " 
                      + game.getAwayTeamScore() + " for " + game.getAwayTeam());
    }
    
    @Override
    public void onScoreChange(Game game){
         scoreChanges++;
    }
}
```


Now, let's define the **PDF Summary observer**.
```java
public class PDFGeneratorObserver implements GameObserver {
    private final PdfGenerator<GameReportStyle> pdfGen = new PdfGenerator<>();

	@Override
    public void onGameStart(Game game){
        pdfGen.addTitle(game.getHomeTeam() + " versus " + game.getAwayTeam());
        pdfGen.addParagraph("Game starts");
    }

	@Override
    public void onGameStop(Game game){
        pdfGen.addParagraph("Game ends between " + game.getHomeTeam() + " and " + game.getAwayTeam());
        pdfGen.addParagraph("Final score: " + game.getHomeTeamScore() + "/" + game.getAwayTeamScore());
    }

	@Override
    public void onScoreChange(Game game){
    	pdfGen.addParagraph("Score changes: " + game.getHomeTeamScore() + "/" + game.getAwayTeamScore());
    }

	/* Our observer can define it's own method as well! */
    public void saveReportInFile(Path pdfFile) throws IOException {
    	pdfGen.writeIn(pdfFile);
    }
}
```


****
## Push vs Pull

In our example, we created **pull observers**
	*Upon notification, **they retrieve information from the observable object***
However, it is also possible to **pass information while notifying**. They become **push observers**.
	*We de-couples codes by pushing the values from an object to another one*

When choosing pull, we **give more flexibility to the observers**, but we **must make methods public** so the observer can collect the information by itself (what we did above)
	*This makes the encapsulation weaker*
When choosing push, we **must construct all information the observer needs**
	*More costly and less flexible, but encapsulation is fine*


**Both solutions are reliable, just chose the correct one depending on the context**

