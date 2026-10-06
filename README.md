/*
    ===========================================================
     STUDENT MANAGEMENT SYSTEM (Console-based, C++)
    ===========================================================
    Features:
      1. Add Student
      2. Display All Students
      3. Search Student (by Roll Number)
      4. Update Student
      5. Delete Student
      6. Exit

    Data is stored persistently in a binary file "students.dat"
    using file handling (fstream), so records remain saved
    even after the program is closed and reopened.
    ===========================================================
*/

#include <iostream>
#include <fstream>
#include <iomanip>
#include <cstring>
#include <limits>

using namespace std;

struct Student {
    int rollNo;
    char name[50];
    int age;
    char course[30];
    float marks;
};

const char* FILENAME = "students.dat";

void clearInputBuffer() {
    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

void pauseScreen() {
    cout << "\nPress Enter to continue...";
    cin.get();
}

bool rollExists(int rollNo) {
    ifstream fin(FILENAME, ios::binary);
    Student s;
    while (fin.read(reinterpret_cast<char*>(&s), sizeof(Student))) {
        if (s.rollNo == rollNo) {
            fin.close();
            return true;
        }
    }
    fin.close();
    return false;
}

void addStudent() {
    Student s;

    cout << "\n----- Add New Student -----\n";
    cout << "Enter Roll Number: ";
    while (!(cin >> s.rollNo)) {
        cout << "Invalid input. Enter numeric Roll Number: ";
        clearInputBuffer();
    }
    clearInputBuffer();

    if (rollExists(s.rollNo)) {
        cout << "A student with Roll Number " << s.rollNo
             << " already exists. Use Update instead.\n";
        pauseScreen();
        return;
    }

    cout << "Enter Name: ";
    cin.getline(s.name, 50);

    cout << "Enter Age: ";
    while (!(cin >> s.age)) {
        cout << "Invalid input. Enter numeric Age: ";
        clearInputBuffer();
    }
    clearInputBuffer();

    cout << "Enter Course: ";
    cin.getline(s.course, 30);

    cout << "Enter Marks: ";
    while (!(cin >> s.marks)) {
        cout << "Invalid input. Enter numeric Marks: ";
        clearInputBuffer();
    }
    clearInputBuffer();

    ofstream fout(FILENAME, ios::binary | ios::app);
    if (!fout) {
        cout << "Error opening file for writing.\n";
        return;
    }
    fout.write(reinterpret_cast<char*>(&s), sizeof(Student));
    fout.close();

    cout << "\nStudent record added successfully!\n";
    pauseScreen();
}

void displayAllStudents() {
    ifstream fin(FILENAME, ios::binary);
    if (!fin) {
        cout << "\nNo records found. File does not exist yet.\n";
        pauseScreen();
        return;
    }

    Student s;
    bool found = false;

    cout << "\n----------------------------------------------------------------\n";
    cout << left << setw(10) << "Roll No" << setw(20) << "Name"
         << setw(6) << "Age" << setw(15) << "Course" << setw(8) << "Marks" << "\n";
    cout << "----------------------------------------------------------------\n";

    while (fin.read(reinterpret_cast<char*>(&s), sizeof(Student))) {
        found = true;
        cout << left << setw(10) << s.rollNo
             << setw(20) << s.name
             << setw(6) << s.age
             << setw(15) << s.course
             << setw(8) << s.marks << "\n";
    }
    fin.close();

    if (!found) {
        cout << "No student records available.\n";
    }
    cout << "----------------------------------------------------------------\n";
    pauseScreen();
}

void searchStudent() {
    int rollNo;
    cout << "\nEnter Roll Number to search: ";
    while (!(cin >> rollNo)) {
        cout << "Invalid input. Enter numeric Roll Number: ";
        clearInputBuffer();
    }
    clearInputBuffer();

    ifstream fin(FILENAME, ios::binary);
    Student s;
    bool found = false;

    while (fin.read(reinterpret_cast<char*>(&s), sizeof(Student))) {
        if (s.rollNo == rollNo) {
            found = true;
            cout << "\n----- Student Found -----\n";
            cout << "Roll No : " << s.rollNo << "\n";
            cout << "Name    : " << s.name << "\n";
            cout << "Age     : " << s.age << "\n";
            cout << "Course  : " << s.course << "\n";
            cout << "Marks   : " << s.marks << "\n";
            break;
        }
    }
    fin.close();

    if (!found) {
        cout << "\nNo student found with Roll Number " << rollNo << ".\n";
    }
    pauseScreen();
}

void updateStudent() {
    int rollNo;
    cout << "\nEnter Roll Number of student to update: ";
    while (!(cin >> rollNo)) {
        cout << "Invalid input. Enter numeric Roll Number: ";
        clearInputBuffer();
    }
    clearInputBuffer();

    fstream file(FILENAME, ios::binary | ios::in | ios::out);
    if (!file) {
        cout << "\nNo records found.\n";
        pauseScreen();
        return;
    }

    Student s;
    bool found = false;

    while (file.read(reinterpret_cast<char*>(&s), sizeof(Student))) {
        if (s.rollNo == rollNo) {
            found = true;

            cout << "\nCurrent Details:\n";
            cout << "Name    : " << s.name << "\n";
            cout << "Age     : " << s.age << "\n";
            cout << "Course  : " << s.course << "\n";
            cout << "Marks   : " << s.marks << "\n";

            cout << "\nEnter New Name: ";
            cin.getline(s.name, 50);

            cout << "Enter New Age: ";
            while (!(cin >> s.age)) {
                cout << "Invalid input. Enter numeric Age: ";
                clearInputBuffer();
            }
            clearInputBuffer();

            cout << "Enter New Course: ";
            cin.getline(s.course, 30);

            cout << "Enter New Marks: ";
            while (!(cin >> s.marks)) {
                cout << "Invalid input. Enter numeric Marks: ";
                clearInputBuffer();
            }
            clearInputBuffer();

            int pos = -static_cast<int>(sizeof(Student));
            file.seekp(pos, ios::cur);
            file.write(reinterpret_cast<char*>(&s), sizeof(Student));

            cout << "\nStudent record updated successfully!\n";
            break;
        }
    }
    file.close();

    if (!found) {
        cout << "\nNo student found with Roll Number " << rollNo << ".\n";
    }
    pauseScreen();
}

void deleteStudent() {
    int rollNo;
    cout << "\nEnter Roll Number of student to delete: ";
    while (!(cin >> rollNo)) {
        cout << "Invalid input. Enter numeric Roll Number: ";
        clearInputBuffer();
    }
    clearInputBuffer();

    ifstream fin(FILENAME, ios::binary);
    if (!fin) {
        cout << "\nNo records found.\n";
        pauseScreen();
        return;
    }

    ofstream fout("temp.dat", ios::binary);
    Student s;
    bool found = false;

    while (fin.read(reinterpret_cast<char*>(&s), sizeof(Student))) {
        if (s.rollNo == rollNo) {
            found = true;
            continue;
        }
        fout.write(reinterpret_cast<char*>(&s), sizeof(Student));
    }
    fin.close();
    fout.close();

    remove(FILENAME);
    rename("temp.dat", FILENAME);

    if (found) {
        cout << "\nStudent record deleted successfully!\n";
    } else {
        cout << "\nNo student found with Roll Number " << rollNo << ".\n";
    }
    pauseScreen();
}

void showMenu() {
    cout << "\n==============================================\n";
    cout << "         STUDENT MANAGEMENT SYSTEM\n";
    cout << "==============================================\n";
    cout << "1. Add Student\n";
    cout << "2. Display All Students\n";
    cout << "3. Search Student\n";
    cout << "4. Update Student\n";
    cout << "5. Delete Student\n";
    cout << "6. Exit\n";
    cout << "==============================================\n";
    cout << "Enter your choice: ";
}

int main() {
    int choice;

    do {
        showMenu();
        while (!(cin >> choice)) {
            cout << "Invalid input. Enter a number between 1-6: ";
            clearInputBuffer();
        }
        clearInputBuffer();

        switch (choice) {
            case 1: addStudent(); break;
            case 2: displayAllStudents(); break;
            case 3: searchStudent(); break;
            case 4: updateStudent(); break;
            case 5: deleteStudent(); break;
            case 6: cout << "\nExiting program. Thank you!\n"; break;
            default: cout << "\nInvalid choice. Please try again.\n"; pauseScreen();
        }

    } while (choice != 6);

    return 0;
}
