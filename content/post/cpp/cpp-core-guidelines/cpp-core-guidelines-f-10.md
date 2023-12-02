---
title: "C++ Core Guidelines F.10 注解"
date: 2023-11-19T04:26:27+08:00
description: "C++ Core Guidelines F.10 注解"
featured: false
toc: true
usePageBundles: false
featureImage: "/images/cpp-core-guidelines.png"
featureImageAlt: 'C++ Core Guidelines'
featureImageCap: 'source: https://github.com/isocpp/CppCoreGuidelines'
thumbnail: "/images/iso_cpp_logo.png"
shareImage: "/images/iso_cpp_logo.png"
codeLineNumbers: true
figurePositionShow: true
categories:
  - C++
tags:
  - C++ Best Practices
  - C++ Core Guidelines
comment: true
---

# F.10: If an operation can be reused, give it a name

## Reason

Documentation, readability, opportunity for reuse.

## Example

```c++
struct Rec {
    string name;
    string addr;
    int id;         // unique identifier
};

bool same(const Rec& a, const Rec& b)
{
    return a.id == b.id;
}

std::vector<Rec*> find_id(const std::string& name);    // find all records for "name"

auto x = find_if(vr.begin(), vr.end(),
    [&](Rec& r) {
        if (r.name.size() != n.size()) return false; // name to compare to is in n
        for (int i = 0; i < r.name.size(); ++i)
            if (tolower(r.name[i]) != tolower(n[i])) return false;
        return true;
    }
);
```

There is a useful function lurking here (case insensitive string comparison), as there often is when lambda arguments get large.

```c++
bool compare_insensitive(const string& a, const string& b) {
    if (a.size() != b.size()) return false;
    for (int i = 0; i < a.size(); ++i) if (tolower(a[i]) != tolower(b[i])) return false;
    return true;
}

auto x = find_if(vr.begin(), vr.end(),
    [&](Rec& r) {
        return compare_insensitive(r.name, n);
    }
);
```

Or maybe (if you prefer to avoid the implicit name binding to n):

```c++
auto cmp_to_n = [&n](const string& a) {
    return compare_insensitive(a, n);
};

auto x = find_if(vr.begin(), vr.end(),
    [](const Rec& r) {
        return cmp_to_n(r.name);
    }
);
```

## Note

whether functions, lambdas, or operators.

## Exception

- Lambdas logically used only locally, such as an argument to `for_each` and similar control flow algorithms.
- Lambdas as initializers.

## Enforcement

- (hard) flag similar lambdas
