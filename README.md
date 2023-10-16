Simple multiplayer implementation based on https://www.youtube.com/watch?v=K62jDMLPToA

It uses the high level multiplayer api

# Special multiplayer nodes

The basic_player scene has a MultiplayerSynchronizer node.
* Its function is to syncronize data over the network
* Its root path is set to the basic_player root node -- so it syncronizes data about the basic_player node tree
* It is set to syncronize hte the basic_player:position -- so that we can see the players move around

The Multiplayer_Test scene has a MultiplayerSpawner node.
* Its function is to spawn (create) nodes requred by the multiplayer game.
* Its spawn path is set to the mMultiplayer_Test root node. 
* Its auto spawn list has one element: the basic_player scene.
* So it spawns player (scenes) inside the multplayer (game) scene

# Code

##  multiplayer_test.gd

Almost all multiplayer specific code is set up in multiplayer_test.gd

```gd
extends Node2D
 
var peer = ENetMultiplayerPeer.new()
@export var player_scene: PackedScene
 
 
func _on_host_pressed():
	peer.create_server(1337)
	multiplayer.multiplayer_peer = peer
	multiplayer.peer_connected.connect(_add_player)
	_add_player()
 
func _add_player(id = 1):
	var player = player_scene.instantiate()
	player.name = str(id)
	call_deferred("add_child",player)
 
 
func _on_join_pressed():
	peer.create_client("127.0.0.1", 1337)
	multiplayer.multiplayer_peer = peer

```

## basic_player.gd

The only exception is checking is_multiplayer_authority() in basic_player.gd to make sure that
the player can only control its own character. It is the "authority" of its own character. 

```gd
extends CharacterBody2D
 
func _enter_tree():
	set_multiplayer_authority(name.to_int())
 
func _physics_process(delta):
	if is_multiplayer_authority():
		velocity = Input.get_vector("ui_left","ui_right","ui_up","ui_down") * 400
	move_and_slide()

```

# Note about the local network

The port used is 1337, this must be a free port on the local computer
-- not used by any other server service. If it doesn't work, try to change the port. 

The client ip is set to "127.0.0.1" which is the same as "localhost".