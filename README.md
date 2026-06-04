import os
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

random_questions = [
    {
        "question": "What color is the sky on a clear day? ",
        "answers": ["blue"],
        "success": "Your mind is clear and you gain 1 health.",
        "failure": "You hesitate and lose 1 health.",
    },
    {
        "question": "How many legs does a spider have? ",
        "answers": ["eight", "8"],
        "success": "Sharp thinking rewards you with a golden coin.",
        "failure": "You guess wrong and feel a chill.",
    },
    {
        "question": "What runs but never walks? ",
        "answers": ["water", "a river"],
        "success": "You answer correctly and receive a healing potion.",
        "failure": "The answer slips away and you lose focus.",
    },
]

player = {
    "name": "",
    "character": "",
    "title": "",
    "health": 10,
    "inventory": [],
    "opened_chests": [],
    "used_mountain_boost": False,
    "used_question_shield": False,
}

characters = {
    "warrior": {
        "health": 14,
        "description": "Hardy and strong, ready for danger.",
        "starting_items": ["rope"],
    },
    "ranger": {
        "health": 12,
        "description": "Skilled with navigation and nature.",
        "starting_items": ["map"],
    },
    "mage": {
        "health": 10,
        "description": "Wise and mystical, but fragile.",
        "starting_items": ["lantern"],
    },
}

rooms = {
    "camp": {
        "description": "A quiet camp with a warm fire. Paths lead north to the forest, east to the river, and south to a nearby village.",
        "exits": {"north": "forest", "east": "river", "south": "village"},
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
        "exits": {"west": "camp", "east": "meadow", "north": "lake"},
        "challenge": "cross_river",
    },
    "meadow": {
        "description": "A sunny meadow dotted with wildflowers. The trail branches toward the mountain and a tangled swamp.",
        "exits": {"west": "river", "east": "mountain", "south": "clearing", "north": "swamp"},
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
        "exits": {"west": "meadow", "east": "ruins"},
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
    "village": {
        "description": "A small village with a few cottages and a bustling market. Travelers rest here before heading deeper into the wild.",
        "exits": {"north": "camp", "east": "swamp"},
        "item": "torch",
        "npc": "villager",
    },
    "swamp": {
        "description": "A misty swamp full of twisting paths and hidden dangers.",
        "exits": {"west": "village", "south": "meadow", "east": "lake"},
        "item": "swamp root",
        "challenge": "cross_swamp",
    },
    "lake": {
        "description": "A still lake that reflects the sky like a mirror. Fish leap in the distance.",
        "exits": {"west": "swamp", "east": "tower", "south": "river"},
        "item": "fishing rod",
    },
    "tower": {
        "description": "A tall stone tower that seems to hum with ancient power.",
        "exits": {"west": "lake"},
        "item": "spellbook",
        "npc": "tower guardian",
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
    print(f"Name: {player['title'] if player['title'] else player['name']}")
    if player['character']:
        print(f"Class: {player['character'].title()} - {characters[player['character']]['description']}")
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
    print("""
Map legend: current location is marked with @
""")
    print(build_map())
    print("=" * 40)


def build_map():
    def mark(room):
        label = room.title()
        if room == current_room:
            return f"[@{label}]"
        return f"[{label}]"

    return (
        f"          {mark('forest camp')}\n"
        f"             |\n"
        f"    {mark('forest')} - {mark('cave')}\n"
        f"      |\n"
        f"   {mark('clearing')}\n"
        f"      |\n"
        f"{mark('camp')} - {mark('river')} - {mark('meadow')} - {mark('mountain')} - {mark('ruins')}\n"
        f"      |        \\n"
        f"{mark('village')} - {mark('swamp')} - {mark('lake')} - {mark('tower')}\n"
    )


def show_map():
    print("\n" + build_map())


def ask_random_question():
    question = random.choice(random_questions)
    answer = get_input(question["question"])
    if answer in question["answers"]:
        print(question["success"])
        if "health" in question["success"]:
            player["health"] += 1
        if "coin" in question["success"]:
            player["inventory"].append("golden coin")
        if "potion" in question["success"]:
            player["inventory"].append("healing potion")
    else:
        print(question["failure"])
        if player["character"] == "mage" and not player["used_question_shield"]:
            print("Your magic shields you from any damage this time.")
            player["used_question_shield"] = True
        else:
            player["health"] -= 1


def choose_character():
    print("Choose your character class:")
    for name, data in characters.items():
        print(f"- {name.title()}: {data['description']} (Health {data['health']})")

    while True:
        choice = get_input("Enter warrior, ranger, or mage: ")
        if choice:
            choice = choice.strip().lower()
        if choice in characters:
            player["character"] = choice
            player["health"] = characters[choice]["health"]
            player["inventory"].extend(characters[choice]["starting_items"])
            title = get_input("Enter your adventurer title or nickname: ")
            player["title"] = title.strip().title() if title else "Adventurer"
            print(f"You become {player['title']} the {choice.title()} with {player['health']} health.")
            if player["inventory"]:
                print(f"Starting items: {', '.join(player['inventory'])}")
            return
        print("That is not a valid character. Try again.")


def cross_river():
    print("The current is strong. You need something to help you cross.")
    if "magic stone" in player["inventory"]:
        print("The magic stone glows and calms the river. You cross safely.")
        return True
    print("You try to swim, but the river sweeps you downstream. You lose 2 health.")
    player["health"] -= 2
    return player["health"] > 0


def cross_swamp():
    print("The swamp is thick and treacherous.")
    if player["character"] == "ranger":
        print("Your ranger instincts guide you across safely.")
        return True
    if "rope" in player["inventory"] or "torch" in player["inventory"]:
        print("Using your gear, you push through the swamp and make it across.")
        return True
    print("The swamp pulls at your legs. You lose 1 health.")
    player["health"] -= 1
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
        show_map()
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
    elif current_room == "village":
        if "torch" not in player["inventory"]:
            print("The villager offers you a torch for your journey.")
            player["inventory"].append("torch")
        else:
            print("The villager wishes you good fortune.")
    elif current_room == "tower":
        print("The tower guardian says: 'Only the brave enter the tower with knowledge and heart.'")
    else:
        print(f"The {npc} has nothing more to say.")


def climb_mountain():
    print("The mountain path is steep and rocky.")
    if player["character"] == "warrior" and not player["used_mountain_boost"]:
        print("Your warrior strength carries you safely over the rocks.")
        player["used_mountain_boost"] = True
        return True
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
    choose_character()
    player["name"] = get_input("What is your name, adventurer? ") or "Traveler"
    print(f"Hello, {player['name']} the {player['character'].title()}! Your journey begins now.")

    while True:
        if player["health"] <= 0:
            print("You have fallen on your adventure. Game over.")
            break

        show_status()

        command = get_input("What do you want to do? (move/pickup/use/open/talk/map/question/quit or W/A/S/D) ")
        if command in ("w", "a", "s", "d"):
            direction = {"w": "north", "a": "west", "s": "south", "d": "east"}[command]
            if move(direction):
                if "challenge" in rooms[current_room]:
                    challenge = rooms[current_room]["challenge"]
                    if not globals()[challenge]():
                        print("Your adventure ends here.")
                        break
            continue
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
        elif command == "map":
            show_map()
        elif command == "question":
            ask_random_question()
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
        gui_path = os.path.join(os.path.dirname(__file__), "adventure_game_gui.py")
        subprocess.run([sys.executable, gui_path], check=True)
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
