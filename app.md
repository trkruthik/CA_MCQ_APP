//First, we need to import the necessary libraries for connecting to Google Sheets and creating a UI
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.RadioButton
import android.widget.RadioGroup
import android.widget.TextView
import com.google.api.services.sheets.v4.Sheets
import com.google.api.services.sheets.v4.model.ValueRange

class QuizActivity : AppCompatActivity() {

//Initialize variables for UI elements and question list
private lateinit var questionTextView: TextView
private lateinit var option1RadioButton: RadioButton
private lateinit var option2RadioButton: RadioButton
private lateinit var option3RadioButton: RadioButton
private lateinit var option4RadioButton: RadioButton
private lateinit var radioGroup: RadioGroup
private var questionList = ArrayList<Question>()

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_quiz)

    //Link UI elements to variables
    questionTextView = findViewById(R.id.question_text_view)
    option1RadioButton = findViewById(R.id.option_1_radio_button)
    option2RadioButton = findViewById(R.id.option_2_radio_button)
    option3RadioButton = findViewById(R.id.option_3_radio_button)
    option4RadioButton = findViewById(R.id.option_4_radio_button)
    radioGroup = findViewById(R.id.radio_group)

    //Initialize the Sheets API client and authenticate with Google
    val sheetsService = Sheets.Builder(AndroidHttp.newCompatibleTransport(), GsonFactory(), GoogleAccountCredential.usingOAuth2(this, listOf(SheetsScopes.SPREADSHEETS))).build()

    //Specify the spreadsheet and worksheet to pull data from
    val spreadsheetId = "YOUR_SPREADSHEET_ID"
    val range = "YOUR_WORKSHEET_NAME!A1:Z"

    //Use the Sheets API to retrieve the data from the specified worksheet
    val result = sheetsService.spreadsheets().values().get(spreadsheetId, range).execute()
    val rows = result.getValues()

    //Create a list of multiple choice questions from the retrieved data
    for (row in rows) {
        val question = row[0] as String
        val option1 = row[1] as String
        val option2 = row[2] as String
        val option3 = row[3] as String
        val option4 = row[4] as String
        val correctAnswer = row[5] as String
        questionList.add(Question(question, option1, option2, option3, option4, correctAnswer))
    }

    //Begin the quiz by displaying the first question
    displayQuestion(questionList[0])

    //Listen for the user's answer selection
    radioGroup.setOnCheckedChangeListener { _, checkedId ->
        val userAnswer = when (checkedId) {
            R.id.option_1_radio_button -> option1RadioButton.text.toString()
            R.id.option_2_radio_button -> option2RadioButton.text.toString()
            R.id.option_3_radio_button -> option3RadioButton.text.toString()
            R.id.option_4_radio_button -> option4RadioButton.text.toString()
        else -> ""
        }
    checkAnswer(userAnswer)
    }
  }
private fun displayQuestion(question: Question) {
    questionTextView.text = question.question
    option1RadioButton.text = question.option1
    option2RadioButton.text = question.option2
    option3RadioButton.text = question.option3
    option4RadioButton.text = question.option4
}

private fun checkAnswer(userAnswer: String) {
    val currentQuestion = questionList[0]
    if (userAnswer == currentQuestion.correctAnswer) {
        score++
    }

    //Remove the current question from the list and display the next question
    questionList.removeAt(0)
    if (questionList.isNotEmpty()) {
        displayQuestion(questionList[0])
        radioGroup.clearCheck()
    } else {
        showScore(score)
    }
}

private fun showScore(score: Int) {
    //Code to display the user's score to the user
}


