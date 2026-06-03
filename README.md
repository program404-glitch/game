import sys
import subprocess
import random

riddles = [
    {
        "question": "What has keys but can't open locks? ",
        "answers": ["piano", "a piano"],
        "success": "The chest opens! You find a healing potion and a golden coin.",
        "failure": "The chest remains locked and a bat startles you. You lose 1 health.",
    },
    {
        "question": "I have branches, but no fruit, trunk or leaves. What am I? ",
        "answers": ["bank", "a bank"],
        "success": "The chest opens! You find a healing potion and a golden coin.",
        "failure": "The chest remains locked and a bat startles you. You lose 1 health.",
    },
    {
        "question": "What can travel around the world while staying in a corner? ",
        "answers": ["stamp", "a stamp"],
        "success": "The chest opens! You find a healing potion and a golden coin.",
        "failure": "The chest remains locked and a bat startles you. You lose 1 health.",
    },
]

chest_rewards = {
    "wooden chest": ["map", "golden coin"],
    "hidden chest": ["magic stone"],
    "iron chest": ["healing potion", "rope"],
    "ancient chest": ["ancient scroll", "golden coin"],
}

player = {
    "name": "",
    "health": 10,
    "inventory": [],
    "opened_chests": [],
}

rooms = {
    "camp": {
        "description": "A quiet camp with a warm fire. Paths lead north to the forest and east to the river.",
        "exits": {"north": "forest", "east": "river"},
        "npc": "camp elder",
    },
    "forest": {
        "description": "The forest is dense and full of shadows. You can hear people talking in a hidden forest camp to the west.",
        "exits": {"south": "camp", "east": "cave", "west": "forest camp"},
        "item": "magic stone",
        "npc": "ranger",
        "chest": "wooden chest",
    },
    "forest camp": {
        "description": "A small forest camp with a few tents and a friendly guide. The smell of pine and cooking smoke surrounds you.",
        "exits": {"east": "forest", "north": "clearing"},
        "npc": "forest guide",
        "chest": "hidden chest",
    },
    "river": {
        "description": "A fast river runs through the valley. The current looks strong.",
        "exits": {"west": "camp", "east": "meadow"},
        "challenge": "cross_river",
    },
    "meadow": {
        "description": "A sunny meadow dotted with wildflowers. The mountain trail begins here.",
        "exits": {"west": "river", "north": "mountain", "south": "clearing"},
        "item": "rope",
        "chest": "iron chest",
    },
    "clearing": {
        "description": "A quiet clearing with a lantern hanging from a branch. The air feels calm.",
        "exits": {"east": "forest", "north": "meadow"},
        "item": "lantern",
    },
    "mountain": {
        "description": "A steep mountainside with a narrow path. The air is thin and cold.",
        "exits": {"south": "meadow", "east": "ruins"},
        "challenge": "climb_mountain",
    },
    "ruins": {
        "description": "Ancient stone ruins hidden among the peaks. Strange glyphs glow faintly.",
        "exits": {"west": "mountain"},
        "item": "ancient scroll",
        "challenge": "read_scroll",
        "chest": "ancient chest",
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
    if "chest" in rooms[current_room] and current_room not in player["opened_chests"]:
        print(f"You notice a {rooms[current_room]['chest']} here.")
    if "npc" in rooms[current_room]:
        print(f"You notice a {rooms[current_room]['npc']} here.")
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
    riddle = random.choice(riddles)
    answer = get_input(riddle["question"])
    if answer in riddle["answers"]:
        print(riddle["success"])
        player["inventory"].append("healing potion")
        player["inventory"].append("golden coin")
        return True
    print(riddle["failure"])
    player["health"] -= 1
    return player["health"] > 0


def open_chest():
    current = rooms[current_room]
    if "chest" not in current or current_room in player["opened_chests"]:
        print("There is no chest to open here.")
        return
    print(f"You find a {current['chest']} here. It is locked with a riddle:")
    riddle = random.choice(riddles)
    answer = get_input(riddle["question"])
    if answer in riddle["answers"]:
        print(riddle["success"])
        rewards = chest_rewards.get(current["chest"], ["gold coin"])
        for reward in rewards:
            player["inventory"].append(reward)
            print(f"You receive: {reward}.")
        player["opened_chests"].append(current_room)
        return
    print(riddle["failure"])
    player["health"] -= 1


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
    elif item_name == "lantern":
        print("You hold the lantern high. Dark places will be easier to explore.")
    elif item_name == "rope":
        print("You coil the rope around your pack. It may help you climb or cross rough terrain.")
    elif item_name == "ancient scroll":
        print("You read the ancient scroll. Its words speak of hidden paths and secret magic.")
    elif item_name == "map":
        print("The map shows a path to the forest camp and the hidden clearing.")
    else:
        print(f"You can't use the {item_name} here.")


def talk():
    room = rooms[current_room]
    if "npc" not in room:
        print("There is no one to talk to here.")
        return
    npc = room["npc"]
    print(f"You talk with the {npc}.")
    if current_room == "camp":
        if "healing potion" not in player["inventory"]:
            print("The camp elder gives you a healing potion and wishes you well.")
            player["inventory"].append("healing potion")
        else:
            print("The camp elder says: 'Stay strong and listen to the forest.'")
    elif current_room == "forest":
        print("The ranger warns: 'The cave is dark. A lantern or a calm stone will keep you safe.'")
    elif current_room == "forest camp":
        if "map" not in player["inventory"]:
            print("The forest guide offers you a map of hidden routes.")
            player["inventory"].append("map")
        else:
            print("The forest guide points you toward the clearing and the mountain path.")
    else:
        print(f"The {npc} has nothing more to say.")


def climb_mountain():
    print("The mountain path is steep and rocky.")
    if "rope" in player["inventory"]:
        print("Using the rope, you climb safely and reach the ruins.")
        return True
    print("You slip on loose stones and scrape your arm. You lose 2 health.")
    player["health"] -= 2
    return player["health"] > 0


def read_scroll():
    print("The ancient glyphs glow as you enter.")
    if "ancient scroll" in player["inventory"]:
        print("The scroll reveals a secret: 'Light reveals what darkness hides.'")
    else:
        print("Without the scroll, the ruins feel empty and quiet.")
    return True


def game_loop():
    print("Welcome to the simple Python adventure game!")
    player["name"] = get_input("What is your name, adventurer? ") or "Traveler"
    print(f"Hello, {player['name']}! Your journey begins now.")

    while True:
        if player["health"] <= 0:
            print("You have fallen on your adventure. Game over.")
            break

        show_status()

        command = get_input("What do you want to do? (move/pickup/use/open/talk/quit) ")
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
        elif command.startswith("open"):
            open_chest()
        elif command == "talk":
            talk()
        else:
            print("I don't understand that command.")

    print("Goodbye, brave adventurer.")


def launch_gui():
    try:
        print("Launching graphical interface...")
        subprocess.run([sys.executable, "adventure_game_gui.py"], check=True)
    except (FileNotFoundError, subprocess.CalledProcessError):
        print("Unable to launch the GUI. Running console mode instead.")
        game_loop()


def choose_interface():
    print("Choose interface:")
    print("1. Console")
    print("2. GUI")
    choice = get_input("Enter 1 or 2: ")
    if choice == "2":
        launch_gui()
    else:
        game_loop()


if __name__ == "__main__":
    choose_interface()
22
