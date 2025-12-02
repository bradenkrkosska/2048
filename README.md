#include <iostream>
#include <random>
#include <cctype>

using namespace std;

void clearScreen() {
	cout << "\033[2J\033[1;1H";
}

class Board //Will contain much of the same logic from tictactoe:
{
public:
	int generatedNumber;

	// Creates usable spaces
	int boardSpots[4][4] = {
		{0,0,0,0},
		{0,0,0,0},
		{0,0,0,0},
		{0,0,0,0},
	};

	string cellAppearance(int value) {
		if (value == 0) {
			return "    ";
		}

		switch (value) {
		case 2:
			return "\033[95m 2  \033[0m";
			break;
		case 4:
			return "\033[96m 4  \033[0m";
			break;
		case 8:
			return "\033[94m 8  \033[0m";
			break;
		case 16:
			return "\033[91m 16 \033[0m";
			break;
		case 32:
			return "\033[92m 32 \033[0m";
			break;
		case 64:
			return "\033[93m 64 \033[0m";
			break;
		case 128:
			return "\033[95m128 \033[0m";
			break;
		case 256:
			return "\033[96m256 \033[0m";
			break;
		case 512:
			return "\033[94m512 \033[0m";
			break;
		case 1024:
			return "\033[91m1024\033[0m";
			break;
		case 2048:
			return "\033[93m2048\033[0m";
			break;
		}
	}

	// Prints the physical board
	void gameBoard(int currentHighest) {
		clearScreen();
		cout << "Current High Score: " << cellAppearance(currentHighest) << endl;
		cout << "\033[96m      ========2048=========\033[0m";

		cout << endl << "      " << "\033[34m=====================\033[0m" << endl;
		for (int i = 0; i < 4; i++) {
			cout << "      ";
			cout << "\033[34m|\033[0m";
			for (int j = 0; j < 4; j++) {
				cout << cellAppearance(boardSpots[i][j]) << "\033[34m|\033[0m";
			}
			cout << endl << "      " << "\033[34m=====================\033[0m" << endl;
		}
	};

	// Will decide whether the CPU can make a move;
	void validTile(int rowMove, int colMove, int block) {

		// I did get help from chatGPT on this portion. I was running into the issue where the code would never end because valid tile would never reach a point where checking for defeat could occur.
		if (boardSpots[rowMove][colMove] == 0) {
			boardSpots[rowMove][colMove] = block;
			return;
		} // The basic "place if it immediatly finds a space"

		vector<pair<int,int>> emptyTiles; //pair is a vector type that holds two values. one for column and one for row
		for (int i = 0; i < 4; i++) {
			for (int j = 0; j < 4; j++) {
				if (boardSpots[i][j] == 0) {
					emptyTiles.push_back({i, j});
				}
			}
		} //This code will try and find any open spaces and put them in this vector. This is only for if the initial check fails.

		if (emptyTiles.empty()) {
			return;
		} // If everything is full, will simply end the validTile argument and make sure it does not recurse

		mt19937 rng(random_device{}());
		uniform_int_distribution<> dist(0, emptyTiles.size() - 1);
		pair<int,int> p = emptyTiles[dist(rng)];
		int r = p.first;
		int c = p.second;


		boardSpots[r][c] = block; // Using the random number generator, a number will be chosen based on how many empty tiles there are, randomly filling one.
	}

	// Will give 0 - 9. A 0 - 7 will produce 2, 8 or 9 will produce 4 (which the game will try to place in a spot)
	int randomNumber() {
		mt19937 rng(random_device{}());
		uniform_int_distribution<> dist(0, 9);

		generatedNumber = dist(rng);

		if (generatedNumber >= 0 && generatedNumber < 8) {
			return 2;
		}
		else {
			return 4;
		}
	};

	// Will decide a spot for the CPU to try and place in.
	int spot() {
		mt19937 spot(random_device{}());
		uniform_int_distribution<> dist(0, 3);

		return dist(spot);
	};

	void moveTiles(char input) {
		if (input == 'a') { // left
			for (int i = 0; i < 4; i++) {
				// shift left
				int pos = 0;
				for (int j = 0; j < 4; j++) {
					if (boardSpots[i][j] != 0) {
						boardSpots[i][pos++] = boardSpots[i][j];
					}
				}
				while (pos < 4) boardSpots[i][pos++] = 0;

				// merge left
				for (int j = 0; j < 3; j++) {
					if (boardSpots[i][j] == boardSpots[i][j + 1] && boardSpots[i][j] != 0) {
						mergeTiles(boardSpots[i][j], boardSpots[i][j + 1]);
					}
				}

				// shift left again
				pos = 0;
				for (int j = 0; j < 4; j++) {
					if (boardSpots[i][j] != 0) {
						boardSpots[i][pos++] = boardSpots[i][j];
					}
				}
				while (pos < 4) boardSpots[i][pos++] = 0;
			}
		}

		else if (input == 'd') { // right
			for (int i = 0; i < 4; i++) {
				// 1. shift right
				int pos = 3;
				for (int j = 3; j >= 0; j--) {
					if (boardSpots[i][j] != 0) {
						boardSpots[i][pos--] = boardSpots[i][j];
					}
				}
				while (pos >= 0) boardSpots[i][pos--] = 0;

				// 2. merge right
				for (int j = 3; j > 0; j--) {
					if (boardSpots[i][j] == boardSpots[i][j - 1] && boardSpots[i][j] != 0) {
						mergeTiles(boardSpots[i][j], boardSpots[i][j - 1]);
					}
				}

				// 3. shift right again
				pos = 3;
				for (int j = 3; j >= 0; j--) {
					if (boardSpots[i][j] != 0) {
						boardSpots[i][pos--] = boardSpots[i][j];
					}
				}
				while (pos >= 0) boardSpots[i][pos--] = 0;
			}
		}

		else if (input == 'w') { // up
			for (int j = 0; j < 4; j++) {
				// 1. shift up
				int pos = 0;
				for (int i = 0; i < 4; i++) {
					if (boardSpots[i][j] != 0) {
						boardSpots[pos++][j] = boardSpots[i][j];
					}
				}
				while (pos < 4) boardSpots[pos++][j] = 0;

				// 2. merge up
				for (int i = 0; i < 3; i++) {
					if (boardSpots[i][j] == boardSpots[i + 1][j] && boardSpots[i][j] != 0) {
						mergeTiles(boardSpots[i][j], boardSpots[i + 1][j]);
					}
				}

				// 3. shift up again
				pos = 0;
				for (int i = 0; i < 4; i++) {
					if (boardSpots[i][j] != 0) {
						boardSpots[pos++][j] = boardSpots[i][j];
					}
				}
				while (pos < 4) boardSpots[pos++][j] = 0;
			}
		}

		else if (input == 's') { // down
			for (int j = 0; j < 4; j++) {
				// 1. shift down
				int pos = 3;
				for (int i = 3; i >= 0; i--) {
					if (boardSpots[i][j] != 0) {
						boardSpots[pos--][j] = boardSpots[i][j];
					}
				}
				while (pos >= 0) boardSpots[pos--][j] = 0;

				// 2. merge down
				for (int i = 3; i > 0; i--) {
					if (boardSpots[i][j] == boardSpots[i - 1][j] && boardSpots[i][j] != 0) {
						mergeTiles(boardSpots[i][j], boardSpots[i - 1][j]);
					}
				}

				// 3. shift down again
				pos = 3;
				for (int i = 3; i >= 0; i--) {
					if (boardSpots[i][j] != 0) {
						boardSpots[pos--][j] = boardSpots[i][j];
					}
				}
				while (pos >= 0) boardSpots[pos--][j] = 0;
			}
		}
	}

	// Merges appropriate tiles
	void mergeTiles(int &newData, int &oldData) {
		if (newData == oldData && newData != 0) {
			newData *= 2;
			oldData = 0;
		}
	};

	// Checks to see if the game can continue on or if failure has been achieved.
	bool isDefeated() {
		for (int i = 0; i < 4; i++) {
			for (int j = 0; j < 4; j++) {

				// Empty tile means moves still possible
				if (boardSpots[i][j] == 0)
					return false;

				if (j < 3 && boardSpots[i][j] == boardSpots[i][j+1])
					return false;

				if (j > 0 && boardSpots[i][j] == boardSpots[i][j-1])
					return false;

				if (i < 3 && boardSpots[i][j] == boardSpots[i+1][j])
					return false;

				if (i > 0 && boardSpots[i][j] == boardSpots[i-1][j])
					return false;
			}
		}
		return true;
	};

	bool checkVictory() {
		for (int i = 0; i < 4; i++) {
			for (int j = 0; j < 4; j++) {

				if (boardSpots[i][j] == 2048) {
					return true;
				}
			}
		}

		return false;
	}

	bool sameBoard(int a[4][4], int b[4][4]) {
		for (int i = 0; i < 4; i++) {
			for (int j = 0; j < 4; j++) {
				if (a[i][j] != b[i][j])
					return false;
			}
		}
		return true;
	}; // Will be used to make sure if a change is made to the board. Mostly used to make sure defeat is actually achieved

	int highestScore(int previousHighest) {
		for (int i = 0; i < 4; i++) {
			for (int j = 0; j < 4; j++) {
				if (boardSpots[i][j] > previousHighest) {
					previousHighest = boardSpots[i][j];
				}

			}
		}
		return previousHighest;
	}
};

int main()
{
	char keepGoing = 'y';

	while (keepGoing != 'q') {
		int previousHighest = 0;
		char input;

beginGame:

		int previous[4][4];
		bool stop = false;
		Board playGame;

		playGame.validTile(playGame.spot(), playGame.spot(), playGame.randomNumber());
		playGame.validTile(playGame.spot(), playGame.spot(), playGame.randomNumber());
		playGame.gameBoard(previousHighest);

		while (stop == false) {
			previousHighest = playGame.highestScore(previousHighest);

			// Following portion was helped by chatGPT to make sure the code wouldnt be ended prematurely due to an invalid move.
			for (int i = 0; i < 4; i++) {
				for (int j = 0; j < 4; j++) {
					previous[i][j] = playGame.boardSpots[i][j];
				}
			}

			playGame.gameBoard(previousHighest);
			if (tolower(input) == 'r') {
				cout << "\t  Game Reseted!" << endl;
			}

			cout << "Controls: " << endl << "Move Tiles: W,A,S,D" << endl << "Quit: Q" << endl << "Reset: R" << endl;
			cin >> input;

			if (tolower(input) == 'q') {
				playGame.gameBoard(previousHighest);
				cout << "Thank you for playing! Should of seen it through though. Get out.";
				goto endGame;
			}

			if (tolower(input) == 'r') {
				goto beginGame;
			}

			playGame.moveTiles(tolower(input));

			if (playGame.checkVictory() == true) {
				playGame.gameBoard(previousHighest);;
				cout << "You won! Congrats, that is better than I could ever do." << endl;
				goto endGame;
			}

			if (playGame.sameBoard(previous, playGame.boardSpots)) {
				continue;
			}

			playGame.validTile(playGame.spot(), playGame.spot(), playGame.randomNumber());

			if (playGame.isDefeated() == true) {
				playGame.gameBoard(previousHighest);;
				cout << "You lose, loser." << endl << "Would you like to play again? Press q to quit: ";
				cin >> keepGoing;
				if (keepGoing == 'q') {
					goto endGame;
				}
				else {
					goto beginGame;
				}
			}
		}

	}
endGame:
	return 0;
}
