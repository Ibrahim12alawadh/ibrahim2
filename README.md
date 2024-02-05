
استعملت لغة dart لتنفيذ المشروع
المشروع عبارة عن اعداد ويجيب عنها المستخدم
يمكن ان تكون الأسالة جمع او ناقص او ضرب او قسمه ويمكن اختياراها من خلال المستخدم
في حال لم يضع المستخدم الاجابة الصيحه يعيد الاجابة عن الاسالة 
في حال اجاب الاجابة الصحيحه ينتقل الى السؤال التالي
و يمكن حسابت عدد اخطاء والاجابات الصحيحة و يضهر نسبت المستخدم اذا كانت aاو b وهكذا

الاكواد

import 'dart:async';
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

enum MathOperation { Addition, Subtraction, Multiplication }

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: UserNamePage(),
    );
  }
}

class UserNamePage extends StatefulWidget {
  @override
  _UserNamePageState createState() => _UserNamePageState();
}

class _UserNamePageState extends State<UserNamePage> {
  TextEditingController _nameController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Enter Your Name'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _nameController,
              decoration: InputDecoration(
                labelText: 'Your Name',
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                String userName = _nameController.text.trim();
                if (userName.isNotEmpty) {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => MathGameStart(userName),
                    ),
                  );
                }
              },
              child: Text('Start Game'),
            ),
          ],
        ),
      ),
    );
  }
}

class MathGameStart extends StatefulWidget {
  final String userName;

  MathGameStart(this.userName);

  @override
  _MathGameStartState createState() => _MathGameStartState();
}

class _MathGameStartState extends State<MathGameStart> {
  MathOperation? selectedOperation;
  late SharedPreferences prefs;

  @override
  void initState() {
    super.initState();
    _loadSavedData();
  }

  _loadSavedData() async {
    prefs = await SharedPreferences.getInstance();
    setState(() {
      selectedOperation =
          MathOperation.values[prefs.getInt('selectedOperation') ?? 0];
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text(widget.userName),
            Text('👤'),
          ],
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Select Math Operation:'),
            DropdownButton<MathOperation>(
              value: selectedOperation,
              onChanged: (MathOperation? value) {
                setState(() {
                  selectedOperation = value;
                  prefs.setInt('selectedOperation', value!.index);
                });
              },
              items: MathOperation.values.map((MathOperation operation) {
                return DropdownMenuItem<MathOperation>(
                  value: operation,
                  child: Text(_getOperationDisplayName(operation)),
                );
              }).toList(),
            ),
            ElevatedButton(
              onPressed: () {
                if (selectedOperation != null) {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) =>
                          MathGame(widget.userName, selectedOperation!),
                    ),
                  );
                }
              },
              child: Text('Start Game'),
            ),
            ElevatedButton(
              onPressed: () {
                resetGame();
              },
              child: Text('Reset Game'),
            ),
          ],
        ),
      ),
    );
  }

  void resetGame() {
    setState(() {
      prefs.remove('score_${MathOperation.Addition.index}');
      prefs.remove('score_${MathOperation.Subtraction.index}');
      prefs.remove('score_${MathOperation.Multiplication.index}');
      prefs.remove('i');
      selectedOperation = null;
    });
  }

  String _getOperationDisplayName(MathOperation operation) {
    switch (operation) {
      case MathOperation.Addition:
        return '+';
      case MathOperation.Subtraction:
        return '-';
      case MathOperation.Multiplication:
        return '*';
    }
  }
}

class MathGame extends StatefulWidget {
  final String userName;
  final MathOperation operation;

  MathGame(this.userName, this.operation);

  @override
  _MathGameState createState() => _MathGameState();
}

class _MathGameState extends State<MathGame> {
  Random random = Random();
  int i = 1;
  late int x;
  late int y;
  late int result;
  late List<String> answerOptions;
  int score = 0;
  int totalQuestions = 10;
  int correctAnswers = 0;
  late SharedPreferences prefs;

  @override
  void initState() {
    super.initState();
    _loadSavedData();
  }

  _loadSavedData() async {
    prefs = await SharedPreferences.getInstance();
    setState(() {
      score = prefs.getInt('score_${widget.operation.index}') ?? 0;
      i = prefs.getInt('i') ?? 1;
      generateNumbers();
    });
  }

  void _saveGameData() {
    prefs.setInt('score_${widget.operation.index}', score);
    prefs.setInt('i', i);
  }

  void generateNumbers() {
    x = random.nextInt(12) + 1;
    y = random.nextInt(12) + 1;
    result = 0;
    answerOptions = _generateAnswerOptions();
  }

  List<String> _generateAnswerOptions() {
    List<String> options = [];
    int correctAnswer = getCorrectAnswer();

    if (widget.operation == MathOperation.Subtraction) {
      options.add(correctAnswer.toString());

      for (int i = 0; i < 2; i++) {
        int randomOption;
        do {
          randomOption = getRandomOption();
        } while (options.contains(randomOption.toString()) ||
            randomOption == correctAnswer);
        options.add(randomOption.toString());
      }
      options.add(getRandomNegativeOption().toString());
    } else {
      options = [correctAnswer.toString()];

      for (int i = 0; i < 3; i++) {
        int randomOption;
        do {
          randomOption = getRandomOption();
        } while (options.contains(randomOption.toString()) ||
            randomOption == int.parse(options[0]));
        options.add(randomOption.toString());
      }
    }

    options.shuffle();
    return options;
  }

  int getCorrectAnswer() {
    switch (widget.operation) {
      case MathOperation.Addition:
        return x + y;
      case MathOperation.Subtraction:
        return x - y;
      case MathOperation.Multiplication:
        return x * y;
    }
  }

  int getRandomOption() {
    return random.nextInt(50);
  }

  int getRandomNegativeOption() {
    return -random.nextInt(12) - 1;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text('👤'),
            Text(widget.userName),
            Text(_getOperationDisplayName(widget.operation)),
          ],
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            i <= totalQuestions
                ? Column(
                    children: [
                      Text('$x ${_getOperationSymbol()} $y = ?'),
                      Column(
                        children: answerOptions.map((option) {
                          return ElevatedButton(
                            onPressed: () {
                              result = int.parse(option);
                              checkAnswer();
                            },
                            child: Text('$option'),
                          );
                        }).toList(),
                      ),
                      SizedBox(height: 20),
                      Text('Question: $i / $totalQuestions'),
                      Text('Score: $score / 100'),
                    ],
                  )
                : ElevatedButton(
                    onPressed: () {
                      showResultDialog();
                    },
                    child: Text('Show Result'),
                  ),
          ],
        ),
      ),
    );
  }

  void checkAnswer() {
    if (_isAnswerCorrect()) {
      _showCorrectAnswer();
      correctAnswers++;
      _updateScore();
      _proceedToNextQuestion();
    } else {
      _showTryAgainDialog();
    }
  }

  bool _isAnswerCorrect() {
    int correctAnswer = getCorrectAnswer();
    return result == correctAnswer;
  }

  void _proceedToNextQuestion() {
    Future.delayed(Duration(seconds: 1), () {
      setState(() {
        i++;
      });

      if (i <= totalQuestions) {
        generateNumbers();
      } else {
        showResultDialog();
      }
    });
  }

  void _showCorrectAnswer() {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: Icon(Icons.check, color: Colors.green, size: 50),
          content: Text('Correct Answer!'),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
              },
              child: Text('OK'),
            ),
          ],
        );
      },
    );
  }

  void _showTryAgainDialog() {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: Text('Wrong Answer'),
          content: Text('Try again'),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
                _proceedToNextQuestion();
              },
              child: Text('OK'),
            ),
          ],
        );
      },
    );
  }

  void showResultDialog() {
    String grade = calculateGrade();
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: Text('Quiz Result'),
          content: Column(
            children: [
              Text('Correct Answers: $correctAnswers'),
              Text('Incorrect Answers: ${totalQuestions - correctAnswers}'),
              Text('Score: $score / 100'),
              Text('Grade: $grade'),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
                Navigator.of(context).pop();
              },
              child: Text('OK'),
            ),
          ],
        );
      },
    );
  }

  String calculateGrade() {
    double percentage = (correctAnswers / totalQuestions) * 100;

    if (percentage >= 90) {
      return 'A+';
    } else if (percentage >= 80) {
      return 'B+';
    } else if (percentage >= 70) {
      return 'C+';
    } else if (percentage >= 60) {
      return 'D+';
    } else {
      return 'F';
    }
  }

  void _updateScore() {
    setState(() {
      score = min(totalQuestions * 10, score + 10);
    });
    _saveGameData(); // Save game data after updating score
  }

  String _getOperationSymbol() {
    switch (widget.operation) {
      case MathOperation.Addition:
        return '+';
      case MathOperation.Subtraction:
        return '-';
      case MathOperation.Multiplication:
        return '*';
    }
  }

  String _getOperationDisplayName(MathOperation operation) {
    switch (operation) {
      case MathOperation.Addition:
        return '1';
      case MathOperation.Subtraction:
        return '2';
      case MathOperation.Multiplication:
        return '3';
    }
  }
}


الاكواد التي تكتب في ملف-->pubspec.yaml
لحفظ التقدم في اللعبة

name: helloworld
description: A new Flutter project.

version: 1.0.0+1

environment:
  sdk: ">=2.19.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  shared_preferences: ^2.0.7

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.2

flutter:
  uses-material-design: true


