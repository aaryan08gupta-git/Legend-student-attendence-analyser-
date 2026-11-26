# Legend-student-attendence-analyser-
Student Attendence Analyzer Project
include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_STUDENTS 100
#define NAME_LENGTH 50
#define REQUIRED_PERCENTAGE 75.0

typedef struct {
    int rollNo; 
    char name[NAME_LENGTH];
    int totalClasses;
    int attendedClasses;
} Student;

Student students[MAX_STUDENTS];
int studentCount = 0;

/* Read and validate an integer from the user. */
int readInt(const char *prompt, int minValue, int maxValue) {
    char input[100];
    char *endPointer;
    long value;

    while (1) {
        printf("%s", prompt);

        if (fgets(input, sizeof(input), stdin) == NULL) {
            printf("\nInput ended. Exiting.\n");
            exit(0);
        }

        value = strtol(input, &endPointer, 10);

        while (isspace((unsigned char)*endPointer)) {
            endPointer++;
        }

        if (endPointer == input || *endPointer != '\0' ||
            value < minValue || value > maxValue) {
            printf("Invalid input. Enter a number from %d to %d.\n",
                   minValue, maxValue);
        } else {
            return (int)value;
        }
    }
}

/* Read a student's name, including spaces. */
void readName(char name[], int size) {
    while (1) {
        printf("Enter student name: ");

        if (fgets(name, size, stdin) == NULL) {
            printf("\nInput ended. Exiting.\n");
            exit(0);
        }

        name[strcspn(name, "\n")] = '\0';

        if (strlen(name) == 0) {
            printf("Name cannot be empty. Try again.\n");
        } else {
            return;
        }
    }
}

int findStudentByRoll(int rollNo) {
    int i;

    for (i = 0; i < studentCount; i++) {
        if (students[i].rollNo == rollNo) {
            return i;
        }
    }

    return -1;
}

double calculatePercentage(const Student *student) {
    if (student->totalClasses == 0) {
        return 0.0;
    }

    return (student->attendedClasses * 100.0) / student->totalClasses;
}

void registerStudent(void) {
    int rollNo;

    if (studentCount >= MAX_STUDENTS) {
        printf("Student limit reached. Cannot register more students.\n");
        return;
    }

    printf("\n--- Register Student ---\n");
    rollNo = readInt("Enter roll number: ", 1, 1000000);

    if (findStudentByRoll(rollNo) != -1) {
        printf("A student with this roll number already exists.\n");
        return;
    }

    students[studentCount].rollNo = rollNo;
    readName(students[studentCount].name, NAME_LENGTH);
    students[studentCount].totalClasses = 0;
    students[studentCount].attendedClasses = 0;

    studentCount++;
    printf("Student registered successfully.\n");
}

void markAttendance(void) {
    int i;
    char choiceLine[20];
    char choice;

    if (studentCount == 0) {
        printf("Please register students first.\n");
        return;
    }

    printf("\n--- Mark Attendance ---\n");
    printf("Enter P for Present or A for Absent.\n");

    for (i = 0; i < studentCount; i++) {
        while (1) {
            printf("%s (Roll No. %d): ",
                   students[i].name, students[i].rollNo);

            if (fgets(choiceLine, sizeof(choiceLine), stdin) == NULL) {
                printf("\nInput ended. Exiting.\n");
                exit(0);
            }

            choice = (char)toupper((unsigned char)choiceLine[0]);

            if ((choice == 'P' || choice == 'A') &&
                (choiceLine[1] == '\n' || choiceLine[1] == '\0')) {
                students[i].totalClasses++;

                if (choice == 'P') {
                    students[i].attendedClasses++;
                }
                break;
            }

            printf("Invalid choice. Please enter P or A.\n");
        }
    }

    printf("Attendance for this class has been recorded.\n");
}

void viewStudentAttendance(void) {
    int rollNo;
    int index;
    double percentage;

    if (studentCount == 0) {
        printf("No students registered yet.\n");
        return;
    }

    printf("\n--- View Student Attendance ---\n");
    rollNo = readInt("Enter roll number: ", 1, 1000000);
    index = findStudentByRoll(rollNo);

    if (index == -1) {
        printf("Student not found.\n");
        return;
    }

    percentage = calculatePercentage(&students[index]);

    printf("\nStudent Name      : %s\n", students[index].name);
    printf("Roll Number       : %d\n", students[index].rollNo);
    printf("Total Classes     : %d\n", students[index].totalClasses);
    printf("Classes Attended  : %d\n", students[index].attendedClasses);
    printf("Attendance        : %.2f%%\n", percentage);

    if (students[index].totalClasses == 0) {
        printf("Status            : No attendance recorded yet\n");
    } else if (percentage >= REQUIRED_PERCENTAGE) {
        printf("Status            : SAFE\n");
    } else {
        printf("Status            : AT RISK (below 75%%)\n");
    }
}

void displayStudentsBelow75(void) {
    int i;
    int found = 0;
    double percentage;

    printf("\n--- Students Below 75%% ---\n");

    for (i = 0; i < studentCount; i++) {
        if (students[i].totalClasses > 0) {
            percentage = calculatePercentage(&students[i]);

            if (percentage < REQUIRED_PERCENTAGE) {
                printf("Roll: %d | Name: %s | Attendance: %.2f%%\n",
                       students[i].rollNo, students[i].name, percentage);
                found = 1;
            }
        }
    }

    if (!found) {
        printf("No students are currently below 75%%.\n");
    }
}

void attendanceRecoveryPredictor(void) {
    int rollNo;
    int index;
    int attended;
    int total;
    int requiredClasses;
    int possibleAbsences;
    double percentage;

    if (studentCount == 0) {
        printf("No students registered yet.\n");
        return;
    }

    printf("\n--- Attendance Recovery Predictor ---\n");
    rollNo = readInt("Enter roll number: ", 1, 1000000);
    index = findStudentByRoll(rollNo);

    if (index == -1) {
        printf("Student not found.\n");
        return;
    }

    attended = students[index].attendedClasses;
    total = students[index].totalClasses;

    if (total == 0) {
        printf("No attendance data is available for this student yet.\n");
        return;
    }

    percentage = calculatePercentage(&students[index]);
    printf("%s currently has %.2f%% attendance (%d out of %d classes).\n",
           students[index].name, percentage, attended, total);

    if (percentage < REQUIRED_PERCENTAGE) {
        /* Find x such that (attended + x) / (total + x) >= 0.75. */
        requiredClasses = 3 * total - 4 * attended;

        printf("The student must attend the next %d consecutive class(es)\n",
               requiredClasses);
        printf("without being absent to reach at least 75%% attendance.\n");
    } else {
        /* Find the maximum x such that attended / (total + x) >= 0.75. */
        possibleAbsences = (4 * attended - 3 * total) / 3;

        printf("The student is currently meeting the 75%% requirement.\n");
        printf("They can miss up to %d future class(es) in a row\n",
               possibleAbsences);
        printf("and still remain at or above 75%%, assuming no other classes are added.\n");
    }
}

void classAttendanceSummary(void) {
    int i;
    int recordedStudents = 0;
    int safeCount = 0;
    int riskCount = 0;
    int highestIndex = -1;
    double percentage;
    double sumOfPercentages = 0.0;

    if (studentCount == 0) {
        printf("No students registered yet.\n");
        return;
    }

    printf("\n--- Class Attendance Summary ---\n");

    for (i = 0; i < studentCount; i++) {
        if (students[i].totalClasses > 0) {
            percentage = calculatePercentage(&students[i]);
            sumOfPercentages += percentage;
            recordedStudents++;

            if (percentage >= REQUIRED_PERCENTAGE) {
                safeCount++;
            } else {
                riskCount++;
            }

            if (highestIndex == -1 ||
                percentage > calculatePercentage(&students[highestIndex])) {
                highestIndex = i;
            }
        }
    }

    printf("Total registered students : %d\n", studentCount);
    printf("Students with records     : %d\n", recordedStudents);
    printf("Students meeting 75%%      : %d\n", safeCount);
    printf("Students below 75%%        : %d\n", riskCount);

    if (recordedStudents > 0) {
        printf("Average individual attendance: %.2f%%\n",
               sumOfPercentages / recordedStudents);
        printf("Highest attendance: %s (%.2f%%)\n",
               students[highestIndex].name,
               calculatePercentage(&students[highestIndex]));
    } else {
        printf("No attendance has been recorded yet.\n");
    }
}

void displayMenu(void) {
    printf("\n========== AttendAI ==========\n");
    printf("1. Register Student\n");
    printf("2. Mark Attendance\n");
    printf("3. View Student Attendance\n");
    printf("4. Display Students Below 75%%\n");
    printf("5. Attendance Recovery Predictor\n");
    printf("6. Class Attendance Summary\n");
    printf("7. Exit\n");
    printf("==============================\n");
}

int main(void) {
    int choice;

    printf("Welcome to AttendAI - Student Attendance Analyzer\n");

    while (1) {
        displayMenu();
        choice = readInt("Enter your choice (1-7): ", 1, 7);

        switch (choice) {
            case 1:
                registerStudent();
                break;
            case 2:
                markAttendance();
                break;
            case 3:
                viewStudentAttendance();
                break;
            case 4:
                displayStudentsBelow75();
                break;
            case 5:
                attendanceRecoveryPredictor();
                break;
            case 6:
                classAttendanceSummary();
                break;
            case 7:
                printf("Thank you for using AttendAI. Goodbye!\n");
                return 0;
        }
    }
}
