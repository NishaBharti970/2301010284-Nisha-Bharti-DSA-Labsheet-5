# 2301010284-Nisha-Bharti-DSA-Labsheet-5
CLASS HEADER
#include <iostream>
#include <vector>
#include <stack>
#include <queue>
#include <string>

class TextEditor {
private:
    std::vector<char> text;             // Array to store text characters
    std::stack<std::string> undoStack;  // Stack for undo operations
    std::stack<std::string> redoStack;  // Stack for redo operations
    std::queue<std::string> clipboard;  // Queue for clipboard management

public:
    void insertText(int position, const std::string& newText);
    void deleteText(int position, int length);
    void undo();
    void redo();
    void copy(int position, int length);
    void paste(int position);
    void printText() const;
};
INSERT
void TextEditor::insertText(int position, const std::string& newText) {
    if (position < 0 || position > text.size()) {
        std::cout << "Invalid position for insertion!" << std::endl;
        return;
    }

    // Insert new text at the specified position
    for (int i = 0; i < newText.size(); ++i) {
        text.insert(text.begin() + position + i, newText[i]);
    }

    // Push the operation to the undo stack
    undoStack.push("delete " + std::to_string(position) + " " + std::to_string(newText.size()));
    // Clear the redo stack after a new operation
    while (!redoStack.empty()) redoStack.pop();
}

DELETE 
void TextEditor::deleteText(int position, int length) {
    if (position < 0 || position + length > text.size()) {
        std::cout << "Invalid position or length for deletion!" << std::endl;
        return;
    }

    // Capture the deleted text
    std::string deletedText;
    for (int i = 0; i < length; ++i) {
        deletedText += text[position + i];
    }

    // Delete the specified characters
    text.erase(text.begin() + position, text.begin() + position + length);

    // Push the operation to the undo stack
    undoStack.push("insert " + std::to_string(position) + " " + deletedText);
    // Clear the redo stack after a new operation
    while (!redoStack.empty()) redoStack.pop();
}

UNDO
void TextEditor::undo() {
    if (undoStack.empty()) {
        std::cout << "Nothing to undo!" << std::endl;
        return;
    }

    std::string operation = undoStack.top();
    undoStack.pop();
    redoStack.push(operation);

    if (operation.find("insert") == 0) {
        // Undo an insert operation
        int position = std::stoi(operation.substr(7, operation.find(' ', 7) - 7));
        int length = std::stoi(operation.substr(operation.find_last_of(' ') + 1));
        text.erase(text.begin() + position, text.begin() + position + length);
    } else if (operation.find("delete") == 0) {
        // Undo a delete operation
        int position = std::stoi(operation.substr(7, operation.find(' ', 7) - 7));
        std::string data = operation.substr(operation.find_last_of(' ') + 1);
        for (int i = 0; i < data.size(); ++i) {
            text.insert(text.begin() + position + i, data[i]);
        }
    }
}

REDO
void TextEditor::redo() {
    if (redoStack.empty()) {
        std::cout << "Nothing to redo!" << std::endl;
        return;
    }

    std::string operation = redoStack.top();
    redoStack.pop();
    undoStack.push(operation);

    if (operation.find("insert") == 0) {
        // Redo an insert operation
        int position = std::stoi(operation.substr(7, operation.find(' ', 7) - 7));
        std::string data = operation.substr(operation.find_last_of(' ') + 1);
        for (int i = 0; i < data.size(); ++i) {
            text.insert(text.begin() + position + i, data[i]);
        }
    } else if (operation.find("delete") == 0) {
        // Redo a delete operation
        int position = std::stoi(operation.substr(7, operation.find(' ', 7) - 7));
        int length = std::stoi(operation.substr(operation.find_last_of(' ') + 1));
        text.erase(text.begin() + position, text.begin() + position + length);
    }
}

COPY PASTE
void TextEditor::copy(int position, int length) {
    if (position < 0 || position + length > text.size()) {
        std::cout << "Invalid position or length for copy!" << std::endl;
        return;
    }

    // Extract the text to copy
    std::string copiedText;
    for (int i = 0; i < length; ++i) {
        copiedText += text[position + i];
    }

    // Add the copied text to the clipboard queue
    clipboard.push(copiedText);
}

void TextEditor::paste(int position) {
    if (clipboard.empty()) {
        std::cout << "Clipboard is empty!" << std::endl;
        return;
    }

    std::string copiedText = clipboard.front();
    clipboard.pop();

    // Paste the text at the specified position
    for (int i = 0; i < copiedText.size(); ++i) {
        text.insert(text.begin() + position + i, copiedText[i]);
    }

    // Push the operation to the undo stack
    undoStack.push("delete " + std::to_string(position) + " " + std::to_string(copiedText.size()));
}




TEXT CASE CODE

#include <iostream>
#include <stack>
#include <queue>
#include <vector>

class TextEditor {
private:
    std::vector<char> text;         // Store the characters of the text editor
    std::stack<std::vector<char>> undoStack;  // Stack for undo operations
    std::stack<std::vector<char>> redoStack;  // Stack for redo operations
    std::queue<std::string> clipboard;  // Queue for clipboard management

public:
    // Insert text at a specified position
    void insertText(int position, const std::string& str) {
        if (position < 0 || position > text.size()) return;  // Check for valid position
        text.insert(text.begin() + position, str.begin(), str.end());
        
        // Save state for undo
        undoStack.push(text);
        while (!redoStack.empty()) redoStack.pop();  // Clear redo stack after new operation
    }

    // Delete text from a specified position and length
    void deleteText(int position, int length) {
        if (position < 0 || position >= text.size()) return;  // Check for valid position
        text.erase(text.begin() + position, text.begin() + position + length);
        
        // Save state for undo
        undoStack.push(text);
        while (!redoStack.empty()) redoStack.pop();  // Clear redo stack after new operation
    }

    // Undo the last operation
    void undo() {
        if (undoStack.empty()) return;  // No operation to undo
        redoStack.push(text);  // Save current state to redo stack
        text = undoStack.top(); // Restore previous state
        undoStack.pop();
    }

    // Redo the last undone operation
    void redo() {
        if (redoStack.empty()) return;  // No operation to redo
        undoStack.push(text);  // Save current state to undo stack
        text = redoStack.top(); // Restore next state from redo stack
        redoStack.pop();
    }

    // Copy text from a specified position and length to the clipboard
    void copy(int position, int length) {
        if (position < 0 || position + length > text.size()) return;  // Check for valid position and length
        clipboard.push(std::string(text.begin() + position, text.begin() + position + length));
    }

    // Paste text from clipboard at a specified position
    void paste(int position) {
        if (position < 0 || position > text.size() || clipboard.empty()) return;  // Check for valid position
        std::string copiedText = clipboard.front();
        clipboard.pop();
        insertText(position, copiedText);  // Insert copied text at the given position
    }

    // Display the current text in the editor
    void displayText() const {
        for (char c : text) {
            std::cout << c;
        }
        std::cout << std::endl;
    }
};

int main() {
    TextEditor editor;

    // Test Case 1: Insert Text
    editor.insertText(0, "Hello");
    std::cout << "After insertText(0, \"Hello\"): ";
    editor.displayText();  // Expected Output: "Hello"

    // Test Case 2: Delete Text
    editor.deleteText(0, 2);
    std::cout << "After deleteText(0, 2): ";
    editor.displayText();  // Expected Output: "llo"

    // Test Case 3: Undo Operation
    editor.undo();
    std::cout << "After undo(): ";
    editor.displayText();  // Expected Output: "Hello"

    // Test Case 4: Redo Operation
    editor.redo();
    std::cout << "After redo(): ";
    editor.displayText();  // Expected Output: "llo"

    // Test Case 5: Copy and Paste
    editor.copy(0, 2);
    editor.paste(5);
    std::cout << "After copy(0, 2), paste(5): ";
    editor.displayText();  // Expected Output: "HelloHe"

    return 0;
}
EXPECTED OUTPUT
After insertText(0, "Hello"): Hello
After deleteText(0, 2): llo
After undo(): llo
After redo(): llo
After copy(0, 2), paste(5): llo
