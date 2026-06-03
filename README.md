import sys

player = {
    "name": "",
    "health": 10,
    "inventory": [],
}

rooms = {
    "camp": {
        "description": "A quiet camp with a warm fire. Paths lead north to the forest and east to the river.",
        "exits": {"north": "forest", "east": "river"},
    },
    "forest": {
        "description": "The forest is dense and full of shadows. You can hear something moving nearby.",
        "exits": {"south": "camp", "east": "cave"},
        "item": "magic stone",
    },
    "river": {
        "description": "A fast river runs through the valley. The current looks strong.",
        "exits": {"west": "camp"},
        "challenge": "cross_river",
    },
    "cave": {
        "description": "A dark cave whose entrance smells of damp earth. You sense treasure inside.",
        "exits": {"west": "forest"},
        "challenge": "find_treasure",
    },
}

current_room = "camp"


def get_input(prompt):
    try:
        return input(prompt).strip().lower()
    except (EOFError, KeyboardInterrupt):
        print("\nGoodbye!")
        sys.exit(0)


def show_status():
    print("\n" + "=" * 40)
    print(f"Location: {current_room.capitalize()}")
    print(rooms[current_room]["description"])
    if "item" in rooms[current_room]:
        item = rooms[current_room]["item"]
        if item not in player["inventory"]:
            print(f"You see a {item} here.")
    print(f"Health: {player['health']}")
    print(f"Inventory: {', '.join(player['inventory']) if player['inventory'] else 'empty'}")
    print("Available exits: " + ", ".join(rooms[current_room]["exits"]))
    print("=" * 40)


def cross_river():
    print("The current is strong. You need something to help you cross.")
    if "magic stone" in player["inventory"]:
        print("The magic stone glows and calms the river. You cross safely.")
        return True
    print("You try to swim, but the river sweeps you downstream. You lose 2 health.")
    player["health"] -= 2
    return player["health"] > 0


def find_treasure():
    print("A treasure chest sits in the cave. It is locked with a riddle:")
    answer = get_input("What has keys but can't open locks? ")
    if answer in ["piano", "a piano"]:
        print("The chest opens! You find a healing potion and a golden coin.")
        player["inventory"].append("healing potion")
        player["inventory"].append("golden coin")
        return True
    print("The chest remains locked and a bat startles you. You lose 1 health.")
    player["health"] -= 1
    return player["health"] > 0


def move(direction):
    global current_room
    if direction in rooms[current_room]["exits"]:
        next_room = rooms[current_room]["exits"][direction]
        current_room = next_room
        return True
    print("You can't go that way.")
    return False


def pickup_item():
    if "item" not in rooms[current_room]:
        print("There is nothing to pick up here.")
        return
    item = rooms[current_room]["item"]
    if item in player["inventory"]:
        print(f"You already picked up the {item}.")
        return
    player["inventory"].append(item)
    print(f"You take the {item}.")


def use_item(item_name):
    if item_name not in player["inventory"]:
        print(f"You don't have a {item_name}.")
        return
    if item_name == "healing potion":
        player["health"] += 5
        player["inventory"].remove(item_name)
        print("You drink the healing potion and restore 5 health.")
    else:
        print(f"You can't use the {item_name} here.")


def game_loop():
    print("Welcome to the simple Python adventure game!")
    player["name"] = get_input("What is your name, adventurer? ") or "Traveler"
    print(f"Hello, {player['name']}! Your journey begins now.")

    while True:
        if player["health"] <= 0:
            print("You have fallen on your adventure. Game over.")
            break

        show_status()

        command = get_input("What do you want to do? (move/pickup/use/quit) ")
        if command == "quit":
            print("Thanks for playing!")
            break
        if command.startswith("move "):
            direction = command.split(" ", 1)[1]
            if move(direction):
                if "challenge" in rooms[current_room]:
                    challenge = rooms[current_room]["challenge"]
                    if not globals()[challenge]():
                        print("Your adventure ends here.")
                        break
        elif command == "pickup":
            pickup_item()
        elif command.startswith("use "):
            item_name = command.split(" ", 1)[1]
            use_item(item_name)
        else:
            print("I don't understand that command.")

    print("Goodbye, brave adventurer.")


if __name__ == "__main__":
    game_loop()
