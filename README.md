#include <iostream>
#include <string>
#include <vector>
#include <sstream>
#include <map>

using namespace std;

struct Grammar {
    string nonTerminal;
    string terminal;
    string startSymbol;
    vector<string> productions;
};

vector<string> split(const string& str, char delimiter) {
    vector<string> tokens;
    stringstream ss(str);
    string token;
    while (getline(ss, token, delimiter)) {
        tokens.push_back(token);
    }
    return tokens;
}

struct Node {
    string value;
    vector<Node*> children;
    ~Node() {
        for (auto child : children) {
            delete child;
        }
    }
};

void printTree(Node* root, int level = 0) {
    if (root == nullptr)
        return;

    for (int i = 0; i < level; i++)
        cout << "  ";

    cout << root->value << endl;

    for (auto child : root->children)
        printTree(child, level + 1);
}

int main() {
    Grammar grammar;
    cout << "Introduzca simbolos no terminales (separados por comas): ";
    getline(cin, grammar.nonTerminal);
    cout << "Ingrese los simbolos terminales (separados por comas) ";
    getline(cin, grammar.terminal);
    cout << "Introduzca el simbolo de arranque: ";
    getline(cin, grammar.startSymbol);
    cout << "Introduzca producciones (separadas por comas): ";
    string productionsStr;
    getline(cin, productionsStr);
    grammar.productions = split(productionsStr, ',');

    // Imprime la gramática
    cout << "Gramatica:" << endl;
    cout << "Simbolos no terminales: " << grammar.nonTerminal << endl;
    cout << "Simbolos terminales: " << grammar.terminal << endl;
    cout << "Simbolo de arranque: " << grammar.startSymbol << endl;
    cout << "Producciones:" << endl;
    for (const auto& production : grammar.productions) {
        cout << production << endl;
    }

    // Construye el árbol de derivación
    map<string, vector<string>> prodMap;
    for (const auto& production : grammar.productions) {
        string lhs = production.substr(0, production.find("->"));
        string rhs = production.substr(production.find("->") + 2);
        prodMap[lhs].push_back(rhs);
    }

    cout << "Ingrese una cadena para generar el arbol de derivacion: ";
    string cadena;
    getline(cin, cadena);

    Node* root = new Node{grammar.startSymbol};
    vector<Node*> currentLevel = {root};
    for (int i = 0; i < cadena.length(); i++) {
        vector<Node*> nextLevel;
        for (auto node : currentLevel) {
            if (node->value.length() == 1 && grammar.nonTerminal.find(node->value[0]) != string::npos) {
                string var = node->value;
                vector<string> productions = prodMap[var];
                for (const auto& prod : productions) {
                    bool allTerminals = true;
                    vector<Node*> newNodes;
                    for (char c : prod) {
                        if (grammar.terminal.find(c) != string::npos) {
                            newNodes.push_back(new Node{string(1, c)});
                        } else if (grammar.nonTerminal.find(c) != string::npos) {
                            allTerminals = false;
                            newNodes.push_back(new Node{string(1, c)});
                        }
                    }
                    if (allTerminals) {
                        for (auto newNode : newNodes) {
                            node->children.push_back(newNode);
                        }
                    } else {
                        Node* prodNode = new Node{prod};
                        for (auto newNode : newNodes) {
                            prodNode->children.push_back(newNode);
                        }
                        node->children.push_back(prodNode);
                        nextLevel.push_back(prodNode);
                    }
                }
            } else {
                nextLevel.push_back(node);
            }
        }
        currentLevel = nextLevel;
    }

    // Imprime el árbol de derivación
    cout << "Arbol de derivacion:" << endl;
    printTree(root);

    delete root;

    return 0;
}
