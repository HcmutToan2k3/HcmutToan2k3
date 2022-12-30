.data
	Board:  .byte '.', '.', '.', '.', '.', '.', '.', 
		      '.', '.', '.', '.', '.', '.', '.',
		      '.', '.', '.', '.', '.', '.', '.',
		      '.', '.', '.', '.', '.', '.', '.',
	   	      '.', '.', '.', '.', '.', '.', '.',
		      '.', '.', '.', '.', '.', '.', '.'
	number_of_column: .asciiz "  1 2 3 4 5 6 7 \n"
	
	space: .asciiz " "
	newLine: .asciiz "\n"
	line: .asciiz "|"
	turn: .byte 'X'
	player_X_choose: .asciiz "Player (X) choose column: "
	player_O_choose: .asciiz "Player (O) choose column: "
	one_to_seven: .asciiz "Invalid move. Please insert a NUMBER between 1 and 7!!\n"
	column_is_filled: .asciiz "Column has been all filled. Please choose another columns!!\n"
	choose_XorO: .asciiz "Choose which symbols to play (X) or (O) ? Press 1 to choose (X)! Press another NUMBER if choose 'O'!!\n"
	O_winner: .asciiz "Congratulations! Player with 'O' has won the game"
	X_winner: .asciiz "Congratulations! Player with 'X' has won the game"
	O_got_undo_1: .asciiz "Player 'O' still got "
	O_got_undo_2: .asciiz " undos left.Press 1 if you want to undo? Press another NUMBER if you dont want to undo? \n"
	O_used_all_undo: .asciiz "Player 'O' already used 3 undos.\n"
	
	draw: .asciiz "The game is DRAW!!"
	
	X_got_undo_1: .asciiz "Player 'X' still got "
	X_got_undo_2: .asciiz " undos left.Press 1 if you want to undo? Press another NUMBER if you dont want to undo? \n"
	X_used_all_undo: .asciiz "Player 'X' already used 3 undos.\n"
	
	X_violate_3_times: .asciiz "'X' has enough 3 violated moves. 'O' has win the game!!"
	O_violate_3_times: .asciiz "'O' has enough 3 violated moves. 'X' has win the game!!"
.text
main:
	li $s4, 3   # undo_X = 3
	li $s5, 3   # undo_O = 3
	li $s6, 3   # violation_X = 3
	li $s7, 3   # violation_O = 3
	jal choose
	jal drawBoard
	jal player_turn
	beqz $s6, X_violate
	beqz $s7, O_violate
	jal drawBoard
	jal player_turn
	beqz $s6, X_violate
	beqz $s7, O_violate
	jal drawBoard
	li $s2, 0
	# s2 = 0 if game not over
	# s2 = 1 if game over
	# s2 = 2 if game draw
	
  Big_Loop:
	jal player_turn
	beqz $s6, X_violate
	beqz $s7, O_violate
	jal drawBoard
	jal undo
	jal drawBoard
	jal game_over
	beqz $s2, Big_Loop
	
  GameOver:
  	li $t1, 2
  	beq $s2, $t1, game_is_draw
	lb $t0, turn
	li $t1, 'X'
	beq $t0, $t1, O_win
	li $v0, 4
	la $a0, X_winner
	syscall
	j exit_all
	
  O_win:
	li $v0, 4
	la $a0, O_winner
	syscall
	j exit_all
	
  game_is_draw:
  	li $v0, 4
  	la $a0, draw
  	syscall
  	j exit_all
  	
  X_violate:	
  	li $v0, 4
  	la $a0, X_violate_3_times
	syscall
	j exit_all
  O_violate:
  
  	li $v0, 4
  	la $a0, O_violate_3_times
	syscall

exit_all:
	li $v0, 10
	syscall
	
	
#choose_XorO
choose:
	
	li $v0, 4
	la $a0, choose_XorO
	syscall
	
	li $v0, 5
	syscall
	move $t1, $v0
	
	li $t0, 1
	beq $t1, $t0, choose_X
	li $t3, 'O'
  	sb $t3, turn
  	j exit_choose
  choose_X:
  exit_choose:
	jr $ra


#draw_board
drawBoard:
	la $a1, Board
	li $t0, 0  # index 
	li $t1, 0  # each_row

  each_row:
  	li $t3, 0
	slti $t2, $t1, 6
	beq $t2, $zero, exit_draw
	addi $t1, $t1, 1
	
	li $v0, 4
	la $a0, line
	syscall
	
	li $v0, 4
	la $a0, space
	syscall
 
  print_row:
  	slti $t4, $t3, 7
  	beq $t4, $zero, exit0
  	addi $t3, $t3, 1
  	
  	lb $t5, ($a1)
  	li $v0, 11
  	move $a0, $t5
  	syscall
  	addi $a1, $a1, 1
  	
  	li $v0, 4
	la $a0, space
	syscall
	
	j print_row
  exit0:
  	li $v0, 4
  	la $a0, line
  	syscall
  	
  	li $v0, 4
	la $a0, newLine
	syscall
	
	j each_row
	
  exit_draw:
  	li $v0, 4
  	la $a0, number_of_column
  	syscall
  	jr $ra
  	

#player_turn
player_turn:
  	la $s0, Board
  	li $t9, '.'
	lb $t0, turn
	li $t1, 'X'
	li $t2, 'O'
	beq $t0, $t1, turn_X
	li $v0, 4
  	la $a0, player_O_choose  #print pompt
  	syscall
  	j turn_O
  turn_X:
  	li $v0, 4
  	la $a0, player_X_choose
  	syscall
  turn_O:
  	li $v0, 5
  	syscall
  	move $t3, $v0  # choice = t3
  	
  input_again:
  	beqz $s6, stop_player_turn
  	beqz $s7, stop_player_turn
  	slti $t4, $t3, 1
  	bne $t4, $zero, insert_again_1to7  #check_1to_7
  	slti $t4, $t3, 8
  	beq $t4, $zero, insert_again_1to7
  	j check_column_filled
  	
  insert_again_1to7:
  	li $v0, 4
  	la $a0, one_to_seven
  	syscall
  	
  	beq $t0, $t1, X_violate_1to7
  	addi $s7, $s7, -1
  	j exit_violation_1to7
  X_violate_1to7:
  	addi $s6, $s6, -1
  exit_violation_1to7:
  	
  	li $v0, 5
  	syscall
  	move $t3, $v0
  	j input_again
  	
  check_column_filled:
  	 
  	add $t4, $s0, $t3
  	addi $t4, $t4, -1
  	lb $t6, ($t4)
  	
  	beq $t6, $t1, column_filled  # board[i] = 'X'
  	beq $t6, $t2, column_filled  # board[i] = 'O'
  	j input_choice
  	
  column_filled:
  	li $v0, 4
  	la $a0, column_is_filled  #check column is filled or not
  	syscall
  	
  	beq $t0, $t1, X_violate_column
  	addi $s7, $s7, -1
  	j exit_violation_column
  X_violate_column:
  	addi $s6, $s6, -1
  	
  exit_violation_column:
  
  	li $v0, 5
  	syscall
  	move $t3, $v0
  	j input_again
  	
  input_choice:
  	li $t5, 1
  	beq $t3, $t5, column_1
  	li $t5, 2
  	beq $t3, $t5, column_2
  	li $t5, 3
  	beq $t3, $t5, column_3
  	li $t5, 4
  	beq $t3, $t5, column_4
  	li $t5, 5
  	beq $t3, $t5, column_5
  	li $t5, 6
  	beq $t3, $t5, column_6
  	li $t5, 7
  	beq $t3, $t5, column_7
  	
	# Do not change $t7
  column_1:
  	addi $t7, $s0, 35
  Loop_1:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_1
  	
  column_2:
  	addi $t7, $s0, 36
  Loop_2:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_2
  	
  column_3:
  	addi $t7, $s0, 37
  Loop_3:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_3
  	
  column_4:
  	addi $t7, $s0, 38
  Loop_4:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_4
  	
  column_5:
  	addi $t7, $s0, 39
  Loop_5:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_5
  	
  column_6:
  	addi $t7, $s0, 40
  Loop_6:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_6
  	
  column_7:
  	addi $t7, $s0, 41
  Loop_7:
  	lb $t6, ($t7)
  	beq $t6, $t9, insert
  	addi $t7, $t7, -7  
  	j Loop_7
  	
  insert:
  	lb $t8, turn	
  	sb $t8, ($t7)
  	
  	beq $t0, $t1, change_to_O
  	sb $t1, turn
  	j stop_player_turn
  	
  change_to_O:
  	sb $t2, turn
  	
  stop_player_turn:
  	jr $ra
  	
  	
#game_over	
game_over:
	la $s0, Board
	li $t0, 'X'
	li $t1, 'O'
	lb $t2, turn
	li $t3, 0   # index = 0
	beq $t0, $t2, check_X
	li $s1, 'X'  # s1 = turn1
	j check
  check_X:
  	li $s1, 'O'  # s1 = turn1

  check:
  	slti $t4, $t3, 42
  	beq $t4, $zero, exit
  	# horizontal 
  	
  	li $t6, 7
  	div $t3, $t6
  	mfhi $t8
  	slti $t9, $t8, 3
  	beq $t9, $zero, vertical
  	lb $t5, ($s0)
  	beq $t5, $s1, check_hori_1
  	j vertical
  	check_hori_1:
  		lb $t5, 1($s0)
  		beq $t5, $s1, check_hori_2
  		j vertical 
  	check_hori_2:
  		lb $t5, 2($s0)
  		beq $t5, $s1, check_hori_3
  		j vertical
  	check_hori_3:
  		lb $t5, 3($s0)
  		beq $t5, $s1, over
  		
  	# vertical
    vertical:
    	lb $t5, ($s0)
  	beq $t5, $s1, check_verti_1
  	j diagonal_1
  	check_verti_1:
  		lb $t5, 7($s0)
  		beq $t5, $s1, check_verti_2
  		j diagonal_1
  	check_verti_2:
  		lb $t5, 14($s0)
  		beq $t5, $s1, check_verti_3
  		j diagonal_1
  	check_verti_3:
  		lb $t5, 21($s0)
  		beq $t5, $s1, over
  		
  	# diagonal from top left -> bottom right
    diagonal_1:
    	li $t6, 7
    	div $t3, $t6
    	mfhi $t8
    	slti $t9, $t8, 3
    	beq $t9, $zero, diagonal_2
    	lb $t5, ($s0)
  	beq $t5, $s1, check_diag_1_1
  	j diagonal_2
  	check_diag_1_1:
  		lb $t5, 8($s0)
  		beq $t5, $s1, check_diag_1_2
  		j diagonal_2
  	check_diag_1_2:
  		lb $t5, 16($s0)
  		beq $t5, $s1, check_diag_1_3
  		j diagonal_2
  	check_diag_1_3:
  		lb $t5, 24($s0)
  		beq $t5, $s1, over
  		
  	# diagonal from top right -> bottom left 
    diagonal_2:
    	lb $t5, ($s0)
    	li $t6, 7
    	div $t3, $t6
    	mfhi $t8
    	slti $t9, $t8, 4
    	bne $t9, $zero, not_over
  	beq $t5, $s1, check_diag_2_1
  	j not_over
  	check_diag_2_1:
  		lb $t5, 6($s0)
  		beq $t5, $s1, check_diag_2_2
  		j not_over
  	check_diag_2_2:
  		lb $t5, 12($s0)
  		beq $t5, $s1, check_diag_2_3
  		j not_over
  	check_diag_2_3:
  		lb $t5, 18($s0)
  		beq $t5, $s1, over
  		
  not_over:
  	addi $s0, $s0, 1
  	addi $t3, $t3, 1
  	j check
  over:
  	li $s2, 1
  	j exit_game_over
  exit:
  	la $s0, Board 
  	li $t1, 0
  check_draw:
  	add $t3, $s0, $t1
  	li $t4, '.'
  	lb $t5, ($t3)
  	beq $t5, $t4, game_not_draw
  	addi $t1, $t1, 1
  	slti $t2, $t1, 7
  	bne $t2, $zero, check_draw
  	li $s2, 2
  	j exit_game_over
  	
  game_not_draw:
  	li $s2, 0
  	
  exit_game_over:
  	jr $ra
  	
  	
#undo 
undo:
	lb $t0, turn
	li $t1, 'X'
	li $t2, 'O'
	li $t9, '.'
	beq $t0, $t1, undo_O_or_not
	j undo_X_or_not
	
  undo_O_or_not:
  	slti $t3, $s5, 1
  	beq $t3, $zero, undo_O
  	li $v0, 4
  	la $a0, O_used_all_undo
  	syscall
  	j exit_undo
  	
  undo_O:
  	#print check many undo left
  	li $v0, 4
  	la $a0, O_got_undo_1
  	syscall
  	li $v0, 1
  	move $a0, $s5
  	syscall
  	li $v0, 4
  	la $a0, O_got_undo_2
  	syscall
  	
  	li $v0, 5
  	syscall
  	move $t4, $v0  #t4 contain 1 if the player want to undo
  	
  	li $t5, 1
  	beq $t4, $t5, execute_undo_O
  	j exit_undo
  execute_undo_O:
  	addi $s5, $s5, -1  
  	sb $t2, turn
  	sb $t9, ($t7)
  	j exit_undo
  	
  undo_X_or_not:
  	slti $t3, $s4, 1
  	beq $t3, $zero, undo_X
  	li $v0, 4
  	la $a0, X_used_all_undo
  	syscall
  	j exit_undo
  	
  undo_X:
  	#print check many undo left
  	li $v0, 4
  	la $a0, X_got_undo_1
  	syscall
  	li $v0, 1
  	move $a0, $s4
  	syscall
  	li $v0, 4
  	la $a0, X_got_undo_2
  	syscall
  	
  	li $v0, 5
  	syscall
  	move $t4, $v0  #t4 contain 1 if the player want to undo
  	
  	li $t5, 1
  	beq $t4, $t5, execute_undo_X
  	j exit_undo
  	
  execute_undo_X:
  	addi $s4, $s4, -1  
  	sb $t1, turn
  	sb $t9, ($t7)
  	j exit_undo
  	
  exit_undo:
  	jr $ra
  	
  	
