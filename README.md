#Project 2: Futoshiki
####By Rihui Zheng and Kyle Lin
####rz1276 and kl3399

##Instructions:

1. Open a command prompt
2. Make sure you have a recent version of python installed
3. Navigate to the directory with `futoshiki.py`
4. Run the python command on `futoshiki.py` followed by all the input file paths as command line arguments
5. For example, depending on which version(s) of python you have, you would run one of the following in the command prompt:
    * `python futoshiki.py Input1.txt Input2.txt Input3.txt`
    * `python3 futoshiki.py Input1.txt Input2.txt Input3.txt`
   

Output1:
```
6 5 3 2 1 4 
5 1 2 3 4 6 
1 6 4 5 2 3 
3 4 6 1 5 2 
2 3 1 4 6 5 
4 2 5 6 3 1
```
Output2:
```
5 1 2 3 6 4 
2 6 1 5 4 3 
4 3 6 1 2 5 
6 2 3 4 5 1 
1 4 5 2 3 6 
3 5 4 6 1 2
```
Output3:
```
6 3 4 1 2 5 
4 1 3 5 6 2 
5 2 6 3 1 4 
3 5 2 6 4 1 
1 4 5 2 3 6 
2 6 1 4 5 3
```

Source Code:
```Python
import sys


def read_info_from_file(file):
    """
    Read and transform the contents of a file
    :Param:
        - file: String representing the file path
    :Return:
        - init_board: 2D list representing the initial board
        - hori_ineq: 2D list representing the positions of horizontal inequalities
        - vert_ineq: 2D list representing the positions of vertical inequalities
    """
    f = open(file, 'r')
    contents = f.read().strip().split('\n\n')
    
    # Parses first 6 lines to a 2D list
    init_board = contents[0].split('\n')
    for i in range(len(init_board)):
        init_board[i] = init_board[i].strip().split(' ')
        for j in range(len(init_board[i])):
            init_board[i][j] = int(init_board[i][j])

    # Parses next 6 lines to a 2D list
    hori_ineq = contents[1].strip().split('\n')
    for i in range(len(hori_ineq)):
        hori_ineq[i] = hori_ineq[i].split(' ')

    # Parses last 5 lines to a 2D list
    vert_ineq = contents[2].strip().split('\n')
    for i in range(len(vert_ineq)):
        vert_ineq[i] = vert_ineq[i].split(' ')
    
    return init_board, hori_ineq, vert_ineq

    
def write_to_file(file, content):
    """
    Writes content to a file
    """
    f = open(file, 'w')
    f.write(content)
    f.close()


def board_to_str(board):
    """
    Returns a 2D list nicely as a string
    """
    board_str = ""
    for row in board:
        for elem in row:
            board_str += str(elem) + " "
        board_str += "\n"

    return board_str[:-1]


class State:
    """
    A class to help organize all the information that the backtracking algorithm needs
    """
    def __init__(self, new_board, new_hori, new_vert, new_domain):
        self.board = new_board
        self.hori_ineq = new_hori
        self.vert_ineq = new_vert
        self.domain = new_domain

    def print_details(self):
        """
        Prints the current board, for debugging purposes
        """
        print("Board:")
        print(board_to_str(self.board))

    def get_rv_and_degree(self, row, col):
        """
        Gets the remaining legal values and degree heuristic of a slot
        :Param: The rows and columns of a certain slot
        :Return: - A list of legal moves for the slot
                 - Number of unassigned neighbors for that slot
        """

        unassigned_neighbors = 0

        # Makes a temporary copy of the domain for the slot
        legal_vals = self.domain[row][col].copy()

        # Check horizontal slots and remove illegal values from domain copy
        for c in range(len(self.board[row])):
            if c != col:
                curr_val = self.board[row][c]
                if not curr_val:
                    unassigned_neighbors += 1
                else:
                    if curr_val in legal_vals:
                        legal_vals.remove(curr_val)

                    if col - c == 1:
                        sign = self.hori_ineq[row][c]
                        if sign == '>':
                            legal_vals = [num for num in legal_vals if num < self.board[row][c]]
                            
                        elif sign == '<':
                            legal_vals = [num for num in legal_vals if num > self.board[row][c]]
                    elif c - col == 1:
                        sign = self.hori_ineq[row][col]
                        if sign == '>':
                            legal_vals = [num for num in legal_vals if num > self.board[row][c]]
                        elif sign == '<':
                            legal_vals = [num for num in legal_vals if num < self.board[row][c]]

        # Check vertical slots and remove illegal values from domain copy
        for r in range(len(self.board)):
            if r != row:
                curr_val = self.board[r][col]
                if not curr_val:
                    unassigned_neighbors += 1
                else:
                    if curr_val in legal_vals:
                        legal_vals.remove(curr_val)

                    if r - row == 1:
                        sign = self.vert_ineq[row][col]
                        if sign == '^':
                            legal_vals = [num for num in legal_vals if num < self.board[r][col]]
                        elif sign == 'v':
                            legal_vals = [num for num in legal_vals if num > self.board[r][col]]
                    elif row - r == 1:
                        sign = self.vert_ineq[r][col]
                        if sign == '^':
                            legal_vals = [num for num in legal_vals if num > self.board[r][col]]
                        elif sign == 'v':
                            legal_vals = [num for num in legal_vals if num < self.board[r][col]]

        return legal_vals, unassigned_neighbors


def forward_checking(state):
    """
    Performs forward checking to reduce domains
    :Param: A state whose domain will be reduced with forward checking
    :Return: None
    """

    # Reduces the domain for the entire row of a given slot
    def reduce_row(row, col):
        for i in range(len(state.board[0])):
            if state.board[row][col] in state.domain[row][i] and i != col:
                state.domain[row][i].remove(state.board[row][col])
            if col - i == 1:
                sign = state.hori_ineq[row][i]
                if sign == '>':
                    state.domain[row][i] = [num for num in state.domain[row][i] if num > state.board[row][col]]
                elif sign == '<':
                    state.domain[row][i] = [num for num in state.domain[row][i] if num < state.board[row][col]]
            elif i - col == 1:
                sign = state.hori_ineq[row][col]
                if sign == '>':
                    state.domain[row][i] = [num for num in state.domain[row][i] if num < state.board[row][col]]
                elif sign == '<':
                    state.domain[row][i] = [num for num in state.domain[row][i] if num > state.board[row][col]]

    # Reduces the domain for the entire column of a given slot
    def reduce_col(row, col):
        for i in range(len(state.board)):
            if state.board[row][col] in state.domain[i][col] and i != row:
                state.domain[i][col].remove(state.board[row][col])
            if i - row == 1:
                sign = state.vert_ineq[row][col]
                if sign == '^':
                    state.domain[i][col] = [num for num in state.domain[i][col] if num > state.board[row][col]]
                elif sign == 'v':
                    state.domain[i][col] = [num for num in state.domain[i][col] if num < state.board[row][col]]
            elif row - i == 1:
                sign = state.vert_ineq[i][col]
                if sign == '^':
                    state.domain[i][col] = [num for num in state.domain[i][col] if num < state.board[row][col]]
                elif sign == 'v':
                    state.domain[i][col] = [num for num in state.domain[i][col] if num > state.board[row][col]]
    
    # Reduces the domains of the entire row and col if slot is already filled
    for row in range(len(state.board)):
        for col in range(len(state.board[0])):
            if state.board[row][col]:
                reduce_row(row, col)
                reduce_col(row, col)


def select_unassigned_variable(state):
    """
    Decides which slot should be the next variable
    :Param: state - the current state of the board
    :Return: The row and column of the next slot to be processed
             A list of legal moves for that slot
    """

    least_mrv = None
    most_deg = None
    next_slot = None

    # Loops through every slot
    for row in range(len(state.board)):
        for col in range(len(state.board[0])):

            # If the slot has not been filled
            if not state.board[row][col]:

                # Gets the legal values and degree heuristic for the slot
                curr_mrv, curr_deg = state.get_rv_and_degree(row, col)

                # Updates if the current mrv and degree heuristics beat the old mrv and degree values
                if not next_slot or len(least_mrv) > len(curr_mrv) or (len(least_mrv) == len(curr_mrv) and most_deg < curr_deg):
                    least_mrv = curr_mrv
                    most_deg = curr_deg
                    next_slot = (row, col)

    return next_slot[0], next_slot[1], least_mrv


def prob_solved(state):
    """
    Checks if the board has been solved
    :Param: Current state of the game
    :Return: True if problem is solved, False if not
    """
    for row in state.board:
        if 0 in row:
            return False
    return True


def backtracking(state):
    """
    Backtracking algorithm to solve the problem
    :Param: state - current state of the game
    :Return: The final solution board (2D list) or None if failure
    """

    # If problem is already solved, return the solution
    if prob_solved(state):
        return state.board

    # Select next slot to fill
    row, col, legal_moves = select_unassigned_variable(state)

    # If no legal moves left, return failure
    if not len(legal_moves):
        return None
    
    # Try each legal moves for the slot
    for val in legal_moves:
        state.board[row][col] = val
        result = backtracking(state)

        # If backtracking finds a solution, return it
        if result:
            return result

    # No solution is found for this path, reset slot value and return failure
    state.board[row][col] = 0
    return None
    


def main():

    args = sys.argv[1:]
    file_no = 1

    # Gets filename
    for file_path in args:

        # Parses file and initiates state
        init_board, hori_ineq, vert_ineq = read_info_from_file(file_path)
        init_domains = [[[dom for dom in range(1,7)] for i in range(len(init_board))] for j in range(len(init_board[0]))]
        curr_state = State(init_board, hori_ineq, vert_ineq, init_domains)

        # Reduces domain with forward checking
        forward_checking(curr_state)

        # Runs the backtracking algorithm
        solution = board_to_str(backtracking(curr_state))

        # Removes the last newline
        solution = solution[:-1]

        # Writes results to output file
        output_f = "Output" + str(file_no) + ".txt"
        if (solution):
            write_to_file(output_f, solution)
        else:
            write_to_file(output_f, "No solution can be found")

        file_no += 1

if __name__ == "__main__":
    main()
```
