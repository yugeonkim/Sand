// Forset
// For set

// language: cpp26
// author: yugeon kim (kor)
// creation time: 2026-01-03
// version: 1.0.0 beta

#include <iostream>
#include <string>
#include <sstream>
#include <vector>
#include <cstdlib>
#include <cctype>
#include <algorithm>

template<typename... Args>
static inline std::string forset(const std::string str, Args&&... args) noexcept {
    std::vector<std::string> values;
    values.reserve(sizeof...(Args));
    ((values.emplace_back(static_cast<std::ostringstream&&>(std::ostringstream{} << args).str())), ...);

    std::string res, substr;
    size_t i = 0, cnt = 0, index;
    bool closed = false;

    while (i < str.size()) {
        if (str[i] == '{') {
            if (i + 1 < str.size() && str[i+1] == '{') {
                res += '{';
                i += 2;
            } else {
                size_t j = i + 1;

                if (str[j] == '-' || ('0' <= str[j] && str[j] <= '9')) {
                    std::ostringstream oss;
                    oss << str[j];
                    while (j < str.size()) {
                        j++;
                        if ('0' <= str[j] && str[j] <= '9') {
                            oss << str[j];
                        } else {
                            if (oss.str().empty()) {
                                std::cerr << "[ERROR] Forset - empty placeholder\n";
                                std::exit(1);
                            } else {
                                for (const auto& ch : oss.str()) {
                                    if (!('0' <= ch && ch <= '9')) {
                                        if (!(ch == '-' && ch == oss.str()[0])) {
                                            std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                                            std::exit(1);
                                        }
                                    }
                                }
                                break;
                            }
                        }
                    }

                    long tmp = std::stol(oss.str());
                    if (tmp < 0) {
                        index = values.size() + tmp;
                    } else if (tmp < values.size()) {
                        index = tmp;
                    } else {
                        std::cerr << "[ERROR] Forset - placeholder index out of range\n";
                        std::exit(1);
                    }
                } else if (str.substr(j, 3) == "mat") {
                    index = cnt++;
                    j += 3;
                    if (index >= values.size()) {
                        std::cerr << "[ERROR] Forset - not eough arguments for 'mat' placeholder\n";
                        std::exit(1);
                    }
                } else {
                    std::cerr << "[ERROR] Forest - invalid placeholder\n";
                    std::exit(1);
                }

                if (str[j] == '}') {
                    res += values[index];
                    j += values[index].size();
                } else if (str[j] == 's') {
                    std::ostringstream oss;
                    j++;
                    while (j < str.size()) {
                        if (str[j] == '}') {
                            if (oss.str().empty()) {
                                std::cerr << "[ERROR] Forset - not continue placeholder\n";
                                std::exit(1);
                            } else {
                                for (const auto& ch : oss.str()) {
                                    if (!('0' <= ch && ch <= '9')) {
                                        std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                                        std::exit(1);
                                    }
                                }
                                break;
                            }
                        } else if ('0' <= str[j] && str[j] <= '9') {
                            oss << str[j];
                        } else {
                            std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                            std::exit(1);
                        }
                        j++;
                    }

                    long width = std::stol(oss.str());
                    if (width < values[index].size()) {
                        res += values[index];
                    } else {
                        long total_space = width - values[index].size();
                        long left_space = total_space / 2;
                        long right_space = total_space - left_space;
                        res = res + std::string(left_space, ' ') + values[index] + std::string(right_space, ' ');
                    }
                } else if (str.substr(j, 2) == "ls") {
                    j += 2;
                    std::ostringstream oss;
                    while (j < str.size()) {
                        if (str[j] == '}') {
                            if (oss.str().empty()) {
                                std::cerr << "[ERROR] Forset - not continue placeholder\n";
                                std::exit(1);
                            } else {
                                for (const auto& ch : oss.str()) {
                                    if (!('0' <= ch && ch <= '9')) {
                                        std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                                        std::exit(1);
                                    }
                                }
                                break;
                            }
                        } else if ('0' <= str[j] && str[j] <= '9') {
                            oss << str[j];
                        } else {
                            std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                            std::exit(1);
                        }
                        j++;
                    }

                    long width = std::stol(oss.str());
                    if (width < values[index].size()) res += values[index];
                    else res = res + std::string(width - values[index].size(), ' ') + values[index];
                } else if (str.substr(j, 2) == "rs") {
                    j += 2;
                    std::ostringstream oss;
                    while (j < str.size()) {
                        if (str[j] == '}') {
                            if (oss.str().empty()) {
                                std::cerr << "[ERROR] Forset - not continue placeholder\n";
                                std::exit(1);
                            } else {
                                for (const auto& ch : oss.str()) {
                                    if (!('0' <= ch && ch <= '9')) {
                                        std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                                        std::exit(1);
                                    }
                                }
                                break;
                            }
                        } else if ('0' <= str[j] && str[j] <= '9') {
                            oss << str[j];
                        } else {
                            std::cerr << "[ERROR] Forset - invalid character in placeholder\n";
                            std::exit(1);
                        }
                        j++;
                    }

                    long width = std::stol(oss.str());
                    if (width < values[index].size()) res += values[index];
                    else res = res + values[index] + std::string(width - values[index].size(), ' ');
                } else {
                    std::cerr << "[ERROR] Forset - invalid placeholder\n";
                    std::exit(1);
                }

                while (i < str.size() && str[i] != '}') i++;
                i++;
            }
        } else {
            res += str[i++];
        }
    }

    size_t j = 0;
    while ((j = res.find("}", j)) != std::string::npos) {
        if (j + 1 < res.size() && res.substr(j, 2) == "}}") {
            res.replace(j, 2, "}");
            j += 1;
        } else {
            std::cerr << "[ERROR] Forset - unmatched placeholder in format string\n";
            std::exit(1);
        }
    }

    return res;
}

int main(void) {
    std::cout << forset("\"{0ls10}\"", 1) << '\n';
    return 0;
}
