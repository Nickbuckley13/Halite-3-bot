# Halite-3-bot
First go developing code
#!/usr/bin/env python3

# Import the Halite SDK, which will let you interact with the game.
import hlt
from hlt import constants

import random
import logging


# This game object contains the initial game state.
game = hlt.Game()
# Respond with your name.
game.ready("MyPythonBot")
ship_status = {}
x = 0
dir_optin = []

dir_offset = []
a = 1
move1 =()
move2=()

def stirring(data):
    # test the area around the ship for highest level of Halite then returns that direction
    test_result= ()
    dir_values = []
    for test in data:
        test_result = dir_values.append(game_map[test].halite_amount)
       
    t = max(enumerate(dir_values), key=(lambda x: x[1]))[0]       
    if  t == 0 and not game_map[data[t]].is_occupied:
        return 'n'
    elif t == 1 and not game_map[data[t]].is_occupied:
        return 's'
    elif t == 2 and not game_map[data[t]].is_occupied:
        return'e'
    elif t == 3 and not game_map[data[t]].is_occupied:
        return'w'
        pass
    else:
        return 'o'
        
def safer_stirring(data):
 # test the area around the ship for highest level of Halite then returns the position of the direction that is not occupied
    test_result= ()
    dir_values = []
    for test in data:
        test_result = dir_values.append(game_map[test].halite_amount)
       
    t = max(enumerate(dir_values), key=(lambda x: x[1]))[0]       
    if  t == 0 and not game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
    elif t == 1 and not game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
    elif t == 2 and not game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
    elif t == 3 and not game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
        
    elif t == 0 and game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
    elif t == 1 and game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
    elif t == 2 and game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
    elif t == 3 and game_map[data[t]].is_occupied:
        return game_map.naive_navigate(ship,data[t])
        pass
   

        
while True:
    # Get the latest game state.
    game.update_frame()
    # You extract player metadata and the updated map metadata here for convenience.
    me = game.me
    game_map = game.game_map
    
    # A command queue holds all the commands you will run this turn.
    command_queue = []

    for ship in me.get_ships():
       
      
       
        if ship.id not in ship_status:
            ship_status[ship.id] = "exploring"
          
        
        if ship_status[ship.id] == "returning":
            if ship.position == me.shipyard.position:
                ship_status[ship.id] = "exploring"
            else:
                move = game_map.naive_navigate(ship, me.shipyard.position)
                command_queue.append(ship.move(move))
                continue
        elif ship.halite_amount >= constants.MAX_HALITE / 4:
            ship_status[ship.id] = "returning"
        logging.info("Ship {} has {} halite.".format(ship.id, ship.halite_amount))
        
                
       
        if game_map[ship.position].halite_amount < constants.MAX_HALITE / 10 or ship.is_full:
               
            dir_optin = ship.position.get_surrounding_cardinals() # [N S E W]
            #command_queue.append(ship.move(stirring(dir_optin)))
            #move1 = safer_stirring(dir_optin)
            #move2 = game_map.naive_navigate(ship,move1)
            #command_queue.append(ship.move(move2))
            #move1 = safer_stirring(dir_optin)
            #move2 = game_map.naive_navigate(ship,move1)
            command_queue.append(ship.move(safer_stirring(dir_optin)))
        else:
           command_queue.append(ship.stay_still())

    # If you're on the first turn and have enough halite, spawn a ship.
    # Don't spawn a ship if you currently have a ship at port, though.
    if game.turn_number == 1 and me.halite_amount >= constants.SHIP_COST and not game_map[me.shipyard].is_occupied:
        command_queue.append(game.me.shipyard.spawn())
        
    elif game.turn_number >= 100 and game.turn_number <=101 and me.halite_amount >= constants.SHIP_COST and not game_map[me.shipyard].is_occupied:
         command_queue.append(game.me.shipyard.spawn())
    
    elif game.turn_number != 1 and len(me.get_ships()) == 0:
        command_queue.append(game.me.shipyard.spawn())

    # Send your moves back to the game environment, ending this turn.
    game.end_turn(command_queue)
