#include <iostream>
#include <vector>
#include <string>
#include <map>
#include <algorithm>
#include <fstream>

class Document {
public:
    std::string title;
    std::string content;

    Document(const std::string &title, const std::string &content) : title(title), content(content) {}
};

class SearchEngine {
private:
    std::vector<Document> documents;
    std::map<std::string, std::vector<int>> index; // Maps keywords to document indices

public:
    // Load documents from a text file (each document separated by a newline)
    void loadDocuments(const std::string &filename) {
        std::ifstream file(filename);
        std::string line;

        while (std::getline(file, line)) {
            auto pos = line.find('|'); // Assume title and content are separated by '|'
            if (pos != std::string::npos) {
                std::string title = line.substr(0, pos);
                std::string content = line.substr(pos + 1);
                documents.push_back(Document(title, content));
                indexDocument(documents.size() - 1, content);
            }
        }
    }

    // Index the document for keywords
    void indexDocument(int docIndex, const std::string &content) {
        std::transform(content.begin(), content.end(), content.begin(), ::tolower); // Convert to lowercase

        std::string word;
        for (char ch : content) {
            if (isalnum(ch)) {
                word += ch;
            } else {
                if (!word.empty()) {
                    index[word].push_back(docIndex);
                    word.clear();
                }
            }
        }
        if (!word.empty()) {
            index[word].push_back(docIndex);
        }
    }

    // Search for a keyword
    void search(const std::string &keyword) {
        auto it = index.find(keyword);
        if (it != index.end()) {
            std::cout << "Documents containing '" << keyword << "':\n";
            for (int docIndex : it->second) {
                std::cout << " - " << documents[docIndex].title << "\n";
            }
        } else {
            std::cout << "No documents found for: " << keyword << std::endl;
        }
    }
};

int main() {
    SearchEngine engine;

    // Load documents from a file
    engine.loadDocuments("documents.txt");

    // Example search
    std::string query;
    std::cout << "Enter keyword to search: ";
    std::cin >> query;

    // Search the documents
    engine.search(query);

    return 0;
}
