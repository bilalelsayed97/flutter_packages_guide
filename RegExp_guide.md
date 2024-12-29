// Flutter RegExp Comprehensive Guide

// 1. BASICS OF REGEXP IN FLUTTER
// RegExp in Flutter is a class that provides support for regular expressions,
// allowing pattern matching and manipulation of strings.

// Basic RegExp instantiation
void regExpBasics() {
  // Simple pattern matching
  RegExp pattern = RegExp(r'hello');
  String text = 'hello world';
  print(pattern.hasMatch(text));  // true
  
  // Note: The r prefix creates a raw string, treating backslashes literally
}

// 2. SYNTAX AND PATTERNS
void regExpPatterns() {
  // Matching specific characters
  RegExp singleChar = RegExp(r'a');  // Matches single 'a'
  RegExp multipleChars = RegExp(r'[aeiou]');  // Matches any vowel
  
  // Quantifiers
  RegExp zeroOrMore = RegExp(r'a*');  // Matches zero or more 'a's
  RegExp oneOrMore = RegExp(r'a+');   // Matches one or more 'a's
  RegExp zeroOrOne = RegExp(r'a?');   // Matches zero or one 'a'
  RegExp exactCount = RegExp(r'a{3}'); // Matches exactly 3 'a's
  RegExp rangeCount = RegExp(r'a{2,4}'); // Matches 2 to 4 'a's
  
  // Character Classes
  RegExp digits = RegExp(r'\d+');     // Matches one or more digits
  RegExp words = RegExp(r'\w+');      // Matches word characters
  RegExp spaces = RegExp(r'\s+');     // Matches whitespace
  RegExp range = RegExp(r'[a-zA-Z]'); // Matches any letter
  
  // Anchors
  RegExp startsWith = RegExp(r'^start'); // Matches 'start' at beginning
  RegExp endsWith = RegExp(r'end$');     // Matches 'end' at the end
  
  // Special Characters
  RegExp escaped = RegExp(r'\.');     // Matches literal dot
  RegExp alternation = RegExp(r'cat|dog'); // Matches 'cat' or 'dog'
}

// 3. USING REGEXP IN FLUTTER/DART
void regExpMethods() {
  String text = 'Hello World! Hello Flutter!';
  RegExp pattern = RegExp(r'Hello');
  
  // hasMatch - checks if pattern exists
  bool exists = pattern.hasMatch(text);  // true
  
  // allMatches - returns all occurrences
  Iterable<Match> matches = pattern.allMatches(text);
  for (Match match in matches) {
    print('Match found at index: ${match.start}');
  }
  
  // firstMatch - returns first occurrence
  Match? firstMatch = pattern.firstMatch(text);
  
  // matchAsPrefix - matches at start of string
  Match? prefixMatch = pattern.matchAsPrefix(text);
  
  // String replacement methods
  String replaced = text.replaceFirst(pattern, 'Hi');  // Replace first occurrence
  String replacedAll = text.replaceAll(pattern, 'Hi'); // Replace all occurrences
  
  // split - splits string on pattern
  List<String> parts = text.split(pattern);
  
  // splitMapJoin - advanced string transformation
  String transformed = pattern.splitMapJoin(
    text,
    onMatch: (m) => m[0]!.toUpperCase(),
    onNonMatch: (n) => n.toLowerCase()
  );
}

// 4. ADVANCED FEATURES
void advancedFeatures() {
  // Named Groups
  RegExp namedGroups = RegExp(r'(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})');
  Match? match = namedGroups.firstMatch('2024-03-15');
  if (match != null) {
    print('Year: ${match.namedGroup('year')}');
    print('Month: ${match.namedGroup('month')}');
    print('Day: ${match.namedGroup('day')}');
  }
  
  // Unicode and Case Sensitivity
  RegExp caseInsensitive = RegExp(r'hello', caseSensitive: false);
  RegExp unicode = RegExp(r'\u{1F600}'); // Matches 😀
  
  // Greedy vs Lazy Quantifiers
  String html = '<div>Content</div>';
  RegExp greedy = RegExp(r'<.*>');        // Matches entire string
  RegExp lazy = RegExp(r'<.*?>');         // Matches each tag separately
}

// 5. PRACTICAL EXAMPLES
class ValidationExamples {
  // Email validation
  static bool isValidEmail(String email) {
    return RegExp(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
        .hasMatch(email);
  }
  
  // Phone number validation (US format)
  static bool isValidPhone(String phone) {
    return RegExp(r'^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$')
        .hasMatch(phone);
  }
  
  // Date extraction
  static List<String> extractDates(String text) {
    RegExp datePattern = RegExp(r'\d{4}-\d{2}-\d{2}');
    return datePattern
        .allMatches(text)
        .map((match) => match.group(0)!)
        .toList();
  }
  
  // Clean string (remove special characters)
  static String cleanString(String text) {
    return text.replaceAll(RegExp(r'[^\w\s]'), '');
  }
}

// 6. PERFORMANCE CONSIDERATIONS
class RegExpOptimization {
  // Cache compiled RegExp objects for reuse
  static final RegExp emailRegExp = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
  
  // Use appropriate patterns
  static bool isValidEmail(String email) {
    return emailRegExp.hasMatch(email);  // Uses cached RegExp
  }
  
  // Avoid excessive backtracking
  static final RegExp betterPattern = RegExp(r'[a-z]+');  // Simple pattern
  static final RegExp worsePattern = RegExp(r'(a|b|c|d|e)+');  // Complex pattern
}

// 7. COMMON PITFALLS AND SOLUTIONS
class RegExpPitfalls {
  // Mistake: Not escaping special characters
  static final RegExp wrong = RegExp('.');  // Matches any character
  static final RegExp correct = RegExp(r'\.');  // Matches literal dot
  
  // Mistake: Greedy matching in HTML
  static String extractContent(String html) {
    // Wrong: RegExp(r'<div>(.*)</div>')  // Greedy matching
    return RegExp(r'<div>(.*?)</div>')    // Lazy matching
        .firstMatch(html)
        ?.group(1) ?? '';
  }
  
  // Mistake: Not using word boundaries
  static bool isWholeWord(String text, String word) {
    return RegExp(r'\b' + word + r'\b').hasMatch(text);
  }
}

// 8. TESTING AND DEBUGGING
void testRegExp() {
  // Unit testing example
  void testEmailValidation() {
    final validEmails = [
      'test@example.com',
      'user.name@domain.co.uk',
    ];
    
    final invalidEmails = [
      'invalid@email',
      '@nodomain.com',
    ];
    
    for (var email in validEmails) {
      assert(ValidationExamples.isValidEmail(email));
    }
    
    for (var email in invalidEmails) {
      assert(!ValidationExamples.isValidEmail(email));
    }
  }
  
  // Debugging tip: Print match information
  void debugRegExp(RegExp pattern, String text) {
    pattern.allMatches(text).forEach((match) {
      print('Match: ${match.group(0)}');
      print('Start: ${match.start}');
      print('End: ${match.end}');
      print('Groups: ${match.groupCount}');
    });
  }
}
