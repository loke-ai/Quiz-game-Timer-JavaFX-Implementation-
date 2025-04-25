import javafx.animation.KeyFrame;
import javafx.animation.Timeline;
import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.geometry.Pos;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.*;
import javafx.stage.Stage;
import javafx.util.Duration;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Main extends Application {

    private Label questionLabel;
    private Button answerButton1, answerButton2, answerButton3, answerButton4;
    private Label timerLabel;
    private Label scoreLabel;
    private ProgressBar timerBar;
    private int timeSeconds = 10;
    private Timeline timeline;
    private int questionIndex = 0;
    private int score = 0;
    private List<Question> questions;
    private Stage primaryStage;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        this.primaryStage = primaryStage;
        primaryStage.setTitle("Quiz Game");

        // Initialize questions
        questions = createQuestions();
        Collections.shuffle(questions); // Shuffle questions

        // UI elements
        questionLabel = new Label();
        questionLabel.setWrapText(true);
        questionLabel.setStyle("-fx-font-size: 16px;");

        answerButton1 = createAnswerButton();
        answerButton2 = createAnswerButton();
        answerButton3 = createAnswerButton();
        answerButton4 = createAnswerButton();

        timerLabel = new Label("Time: " + timeSeconds);
        timerLabel.setStyle("-fx-font-size: 14px;");

        timerBar = new ProgressBar(1.0);

        scoreLabel = new Label("Score: " + score);
        scoreLabel.setStyle("-fx-font-size: 14px;");

        // Layout
        VBox mainLayout = new VBox(10);
        mainLayout.setPadding(new Insets(20));
        mainLayout.setAlignment(Pos.TOP_CENTER);

        HBox buttonLayout = new HBox(10);
        buttonLayout.setAlignment(Pos.CENTER);
        buttonLayout.getChildren().addAll(answerButton1, answerButton2, answerButton3, answerButton4);

        HBox timerScoreLayout = new HBox(20);
        timerScoreLayout.setAlignment(Pos.CENTER);
        timerScoreLayout.getChildren().addAll(timerLabel, timerBar, scoreLabel);

        mainLayout.getChildren().addAll(questionLabel, buttonLayout, timerScoreLayout);

        // Set up scene
        Scene scene = new Scene(mainLayout, 600, 400);
        primaryStage.setScene(scene);

        loadQuestion();
        primaryStage.show();
    }

    private Button createAnswerButton() {
        Button button = new Button();
        button.setPrefWidth(150);
        button.setWrapText(true);
        button.setOnAction(e -> {
            checkAnswer(button.getText());
        });
        return button;
    }

    private void loadQuestion() {
        if (questionIndex < questions.size()) {
            Question currentQuestion = questions.get(questionIndex);
            questionLabel.setText(currentQuestion.getQuestionText());

            List<String> options = new ArrayList<>(currentQuestion.getOptions());
            Collections.shuffle(options);

            answerButton1.setText(options.get(0));
            answerButton2.setText(options.get(1));
            answerButton3.setText(options.get(2));
            answerButton4.setText(options.get(3));

            startTimer();
        } else {
            endQuiz();
        }
    }

    private void startTimer() {
        timeSeconds = 10;
        timerLabel.setText("Time: " + timeSeconds);
        timerBar.setProgress(1.0);

        if (timeline != null) {
            timeline.stop();
        }

        timeline = new Timeline(
            new KeyFrame(Duration.seconds(1), e -> {
                timeSeconds--;
                timerLabel.setText("Time: " + timeSeconds);
                timerBar.setProgress((double) timeSeconds / 10);

                if (timeSeconds <= 0) {
                    timeline.stop();
                    handleTimeout();
                }
            })
        );
        timeline.setCycleCount(timeSeconds);
        timeline.play();
    }

    private void checkAnswer(String selectedAnswer) {
        timeline.stop();
        Question currentQuestion = questions.get(questionIndex);
        if (selectedAnswer.equals(currentQuestion.getOptions().get(currentQuestion.getCorrectIndex()))) {
            score++;
            scoreLabel.setText("Score: " + score);
            showCorrectAnswerFeedback();
        } else {
            showIncorrectAnswerFeedback();
        }

        questionIndex++;
        loadQuestion();
    }

    private void handleTimeout() {
        questionIndex++;
        loadQuestion();
    }

    private void endQuiz() {
        Alert alert = new Alert(Alert.AlertType.INFORMATION);
        alert.setTitle("Quiz Over");
        alert.setHeaderText(null);
        alert.setContentText("Your final score: " + score + " out of " + questions.size());
        alert.setOnHidden(evt -> {
            primaryStage.close(); // Close the stage after alert is dismissed
        });
        alert.showAndWait();
    }
    private void showCorrectAnswerFeedback() {
        Alert alert = new Alert(Alert.AlertType.INFORMATION);
        alert.setTitle("Correct");
        alert.setHeaderText(null);
        alert.setContentText("You answered correctly!");
        alert.showAndWait();
    }

    private void showIncorrectAnswerFeedback() {
        Alert alert = new Alert(Alert.AlertType.INFORMATION);
        alert.setTitle("Incorrect");
        alert.setHeaderText(null);
        alert.setContentText("That was incorrect.");
        alert.showAndWait();
    }


    private List<Question> createQuestions() {
        List<Question> questionList = new ArrayList<>();

        List<String> options1 = List.of("Paris", "London", "Berlin", "Rome");
        questionList.add(new Question("What is the capital of France?", options1, 0));

        List<String> options2 = List.of("Jupiter", "Mars", "Venus", "Saturn");
        questionList.add(new Question("Which planet is known as the Red Planet?", options2, 1));

        List<String> options3 = List.of("Water", "Carbon Dioxide", "Oxygen", "Nitrogen");
        questionList.add(new Question("What do plants primarily absorb from the atmosphere?", options3, 1));

        List<String> options4 = List.of("Berlin", "Madrid", "Oslo", "Helsinki");
        questionList.add(new Question("What is the capital of Spain?", options4, 1));

         List<String> options5 = List.of("Berlin", "Madrid", "Oslo", "Helsinki");
        questionList.add(new Question("What is the capital of Spain?", options5, 1));
        
        return questionList;
    }

    static class Question {
        private String questionText;
        private List<String> options;
        private int correctIndex;

        public Question(String questionText, List<String> options, int correctIndex) {
            this.questionText = questionText;
            this.options = options;
            this.correctIndex = correctIndex;
        }

        public String getQuestionText() {
            return questionText;
        }

        public List<String> getOptions() {
            return options;
        }

        public int getCorrectIndex() {
            return correctIndex;
        }
    }
}
