"""AI vs AI Simulation using Markov Chain for Tic-Tac-Toe."""
import time
import numpy as np
from flask import Blueprint, render_template, jsonify, request, Response
from game_board import GameBoard
from markov_chain_ai import MarkovChainAI
from config import DIFFICULTY_LEVELS, AI_PIECE, PLAYER_PIECE

# Create a Blueprint instead of a Flask app
ai_simulation_bp = Blueprint('ai_simulation', __name__)

@ai_simulation_bp.route('/ai_simulation')
def index():
    """Render the simulation page."""
    return render_template('ai_simulation.html', difficulty_levels=DIFFICULTY_LEVELS)
    
@ai_simulation_bp.route('/text_simulation')
def text_simulation():
    """Render the text simulation page."""
    return render_template('text_simulation.html', difficulty_levels=DIFFICULTY_LEVELS)

@ai_simulation_bp.route('/start_simulation', methods=['POST'])
def start_simulation():
    """Start a new AI vs AI simulation."""
    # Get difficulty levels for both AIs
    ai1_difficulty = request.json.get('ai1_difficulty', 'medium')
    ai2_difficulty = request.json.get('ai2_difficulty', 'hard')
    
    # Initialize the game
    board = GameBoard()
    
    # Store initial state and setup
    result = {
        'board': board.board,
        'game_over': False,
        'winner': None,
        'move_history': [],
        'ai1_stats': {
            'win': 0,
            'block': 0,
            'fork': 0,
            'center': 0,
            'corner': 0,
            'opposite_corner': 0,
            'two_in_a_row': 0,
            'random': 0
        },
        'ai2_stats': {
            'win': 0,
            'block': 0,
            'fork': 0,
            'center': 0,
            'corner': 0,
            'opposite_corner': 0,
            'two_in_a_row': 0,
            'random': 0
        }
    }
    
    return jsonify(result)

@ai_simulation_bp.route('/next_move', methods=['POST'])
def next_move():
    """Process the next move in the AI simulation."""
    # Get current game state from the request
    current_board = request.json.get('board')
    move_history = request.json.get('move_history', [])
    ai1_stats = request.json.get('ai1_stats')
    ai2_stats = request.json.get('ai2_stats')
    ai1_difficulty = request.json.get('ai1_difficulty', 'medium')
    ai2_difficulty = request.json.get('ai2_difficulty', 'hard')
    # Xác định người chơi hiện tại dựa vào số nước đi đã thực hiện
    current_player = "X" if len(move_history) % 2 == 0 else "O"  # X always goes first
    
    # Recreate the game board from the current state
    board = GameBoard()
    board.board = current_board
    
    # Check if the game is already over
    game_over, winner = board.check_game_over()
    if game_over:
        return jsonify({
            'board': board.board,
            'game_over': True,
            'winner': winner,
            'move_history': move_history,
            'ai1_stats': ai1_stats,
            'ai2_stats': ai2_stats,
'current_move': None,
            'explanation': f"Game is over. Winner: {winner if winner != 'draw' else 'Draw'}"
        })
    
    # Determine which AI's turn it is and make the move
    if current_player == "X":  # AI 1's turn (X)
        ai = MarkovChainAI(board, "X", DIFFICULTY_LEVELS[ai1_difficulty])
        stats_to_update = ai1_stats
    else:  # AI 2's turn (O)
        ai = MarkovChainAI(board, "O", DIFFICULTY_LEVELS[ai2_difficulty])
        stats_to_update = ai2_stats
    
    # Get AI's move
    ai_move, explanation = ai.get_next_move()
    
    # Update strategy stats
    for strategy, count in ai.strategy_stats.items():
        if count > 0:
            stats_to_update[strategy] += count
    
    if ai_move:
        # Make the move
        board.make_move(ai_move[0], ai_move[1], current_player)
        move_history.append({
            'player': current_player,
            'position': ai_move,
            'explanation': explanation
        })
    
    # Check if the game is over after this move
    game_over, winner = board.check_game_over()
    
    return jsonify({
        'board': board.board,
        'game_over': game_over,
        'winner': winner,
        'move_history': move_history,
        'ai1_stats': ai1_stats,
        'ai2_stats': ai2_stats,
        'current_move': {
            'player': current_player,
            'position': ai_move,
            'explanation': explanation
        }
    })

@ai_simulation_bp.route('/run_text_simulation')
def run_text_simulation():
    """Run a complete AI vs AI simulation and return text output."""
    # Get parameters from URL query parameters
    ai1_difficulty = request.args.get('ai1_difficulty', 'medium')
    ai2_difficulty = request.args.get('ai2_difficulty', 'hard')
    num_moves = int(request.args.get('num_moves', '9'))  # Default to max moves in a 3x3 board
    
    # Initialize the game
    board = GameBoard()
    move_history = []
    
    # Set up response stream
    def generate():
        yield "Bắt đầu mô phỏng AI vs AI trong trò chơi Tic-Tac-Toe\n"
        yield f"AI 1 (X): Độ khó {ai1_difficulty}\n"
        yield f"AI 2 (O): Độ khó {ai2_difficulty}\n\n"
        
        # Print initial board
        yield "Bàn cờ ban đầu:\n"
        yield _board_to_text(board) + "\n\n"
        
        game_over = False
        winner = None
        
        # Run simulation for max num_moves or until game over
        for move_num in range(min(num_moves, 9)):  # Maximum 9 moves in 3x3 Tic-Tac-Toe
            # Determine current player
            current_player = "X" if move_num % 2 == 0 else "O"
            
            # Initialize the appropriate AI
            if current_player == "X":
                ai = MarkovChainAI(board, "X", DIFFICULTY_LEVELS[ai1_difficulty])
                yield f"Lượt của AI 1 (X):\n"
            else:
                ai = MarkovChainAI(board, "O", DIFFICULTY_LEVELS[ai2_difficulty])
