// Removed: import 'dart:io' show stdin, stdout; // Not supported by DartPad
import 'dart:math';

/// Represents a single scratch card game.
class ScratchCard {
  // Stores the actual values hidden under the scratch-off layer.
  late List<List<String>> _values;
  // Keeps track of which cells have been revealed (scratched).
  late List<List<bool>> _revealed;
  final int _rows;
  final int _cols;
  final Random _random = Random();
  // Stores counts of revealed symbols to check for win conditions.
  final Map<String, int> _revealedCounts = {};

  /// Creates a new scratch card with the specified number of rows and columns.
  /// Throws an [ArgumentError] if the card doesn't have enough cells for a win.
  ScratchCard(this._rows, this._cols) {
    if (_rows * _cols < 3) {
      throw ArgumentError('Card must have at least 3 cells for a winning condition.');
    }
    _values = List.generate(_rows, (_) => List.generate(_cols, (_) => ''));
    _revealed = List.generate(_rows, (_) => List.generate(_cols, (_) => false));
    _initializeCardValues();
  }

  /// Initializes the card with random values, ensuring a winning combination
  /// of 3 identical symbols is always present.
  void _initializeCardValues() {
    // Possible symbols on the card. 'WIN' is 3 characters, so others are padded.
    List<String> possibleValues = ['10 ', '20 ', '50 ', 'WIN', 'LOSE', 'BONUS'];

    // 1. Choose a symbol that will be the winning symbol (appears 3 times).
    String winningSymbol = possibleValues[_random.nextInt(possibleValues.length)];

    // 2. Select 3 distinct random positions for the winning symbol.
    List<int> allIndices = List.generate(_rows * _cols, (index) => index);
    allIndices.shuffle(_random); // Randomize the order of all cell indices
    List<int> winningIndices = allIndices.sublist(0, 3); // Take the first 3 for the winning symbol

    // 3. Fill the grid with values.
    for (int r = 0; r < _rows; r++) {
      for (int c = 0; c < _cols; c++) {
        int currentIndex = r * _cols + c;
        if (winningIndices.contains(currentIndex)) {
          _values[r][c] = winningSymbol;
        } else {
          // Fill non-winning spots with other random values.
          String value;
          do {
            value = possibleValues[_random.nextInt(possibleValues.length)];
            // Slightly reduce the chance of the 'winningSymbol' appearing
            // in non-winning spots to make it a bit harder/more surprising.
          } while (value == winningSymbol && _random.nextDouble() < 0.3);
          _values[r][c] = value;
        }
      }
    }
  }

  /// Prints the current state of the scratch card to the console.
  /// Hidden cells are shown as '###', revealed cells show their value.
  void displayCard() {
    print('\n----- SCRATCH CARD -----');
    // Build column headers as a single string
    String headerRow = '    '; // Padding for row header
    for (int c = 0; c < _cols; c++) {
      headerRow += '$c   '; // One char for col index, 3 spaces for padding
    }
    print(headerRow);

    // Build each row as a single string
    for (int r = 0; r < _rows; r++) {
      String rowString = '$r   '; // Row header and padding
      for (int c = 0; c < _cols; c++) {
        // Display '###' if hidden, or the padded value if revealed.
        String displayValue = _revealed[r][c] ? _values[r][c].padRight(3) : '###';
        rowString += displayValue;
      }
      print(rowString);
    }
    print('------------------------\n');
  }

  /// Attempts to scratch (reveal) a cell at the given row and column.
  /// Returns `true` if scratching was successful, `false` otherwise (e.g., invalid coordinates or already scratched).
  bool scratch(int row, int col) {
    if (row < 0 || row >= _rows || col < 0 || col >= _cols) {
      print('Invalid coordinates. Please enter row and column within bounds (0-${_rows - 1}, 0-${_cols - 1}).');
      return false;
    }
    if (_revealed[row][col]) {
      print('This spot has already been scratched. Try another one.');
      return false;
    }

    // Reveal the cell, add its value to the revealed counts.
    _revealed[row][col] = true;
    String value = _values[row][col];
    _revealedCounts.update(value, (count) => count + 1, ifAbsent: () => 1);
    print('You revealed: ${value.trim()}'); // Trim to remove potential padding spaces
    return true;
  }

  /// Checks if the player has won by revealing 3 identical symbols.
  /// Returns `true` if a win condition is met, `false` otherwise.
  bool checkWinCondition() {
    for (var count in _revealedCounts.values) {
      if (count >= 3) {
        return true;
      }
    }
    return false;
  }

  /// Returns `true` if all cells on the card have been scratched.
  bool get isGameOver {
    for (int r = 0; r < _rows; r++) {
      for (int c = 0; c < _cols; c++) {
        if (!_revealed[r][c]) {
          return false; // Found an unrevealed cell
        }
      }
    }
    return true; // All cells are revealed
  }

  /// Reveals all remaining hidden cells on the card.
  void revealAll() {
    for (int r = 0; r < _rows; r++) {
      for (int c = 0; c < _cols; c++) {
        _revealed[r][c] = true;
      }
    }
    print('\nAll remaining values revealed!');
    displayCard();
  }
}

void main() {
  print('Welcome to the Dart Scratch Card Game!');
  print('Scratch the card to reveal symbols. Match three identical symbols to win!');
  print('\n--- Running in simulation mode (no interactive input in DartPad) ---\n');

  // Create a 3x3 scratch card
  ScratchCard card = ScratchCard(3, 3);
  final Random mainRandom = Random(); // Random for picking simulated moves

  while (true) {
    card.displayCard();

    // Check for win condition after each scratch (and before prompting for next)
    if (card.checkWinCondition()) {
      print('\n🎉 CONGRATULATIONS! You matched three symbols and won! 🎉');
      card.revealAll(); // Show the full card
      break;
    }

    // Check if all spots are scratched (game over without a win)
    if (card.isGameOver) {
      print('\n😔 All spots scratched! No win this time. Better luck next time! 😔');
      card.revealAll(); // Show the full card
      break;
    }

    // --- DartPad Simulation for input ---
    // Instead of interactive stdin.readLineSync(), we simulate a random scratch.
    List<List<int>> unrevealedCells = [];
    // Accessing private fields for simulation in main. In a larger app,
    // ScratchCard might expose a method like `getUnrevealedCells()`.
    for (int r = 0; r < card._rows; r++) {
      for (int c = 0; c < card._cols; c++) {
        if (!card._revealed[r][c]) {
          unrevealedCells.add([r, c]);
        }
      }
    }

    if (unrevealedCells.isEmpty) {
      // This case should be caught by card.isGameOver, but as a safeguard.
      print('\nNo more unrevealed cells to scratch in simulation.');
      break;
    }

    // Pick a random unrevealed cell
    int randomIndex = mainRandom.nextInt(unrevealedCells.length);
    List<int> move = unrevealedCells[randomIndex];
    int row = move[0];
    int col = move[1];

    print('Simulating scratch at Row: $row, Col: $col...');
    // Attempt to scratch the chosen cell
    card.scratch(row, col);

    // Optional: add a small print to indicate a "turn" passed for clarity in simulation.
    print('(Next simulated move will be made)');
    // No actual delay possible in DartPad without making main an async function
    // and using Future.delayed, which is more complex than needed for this fix.
  }

  print('\nThanks for playing!');
}
