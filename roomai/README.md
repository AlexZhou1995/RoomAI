
#  RoomAI Tutorials

## Summary


There are some basic concepts in RoomAI: Player, Environment, Information and Action. The basic procedure of a competition is shown as follows. All players receive information from env, the current player takes a action, and the env forwards with this action.

<pre>
def compete(env, players):
   '''
   :param env: the game environments
   :param players: the array of players, the last player is a chance player.
   :return: the final scores of this competition
   '''
   for player in players:
        players.reset()
   params = dict()
   params["num_normal_players"] = len(players) - 1
   #len(players)-1 normal players and 1 chance player
   
   infos, public_state, person_states, private_state = env.init(params)
   for i in range(len(players)):
       players[i].receive_info(infos[i])

   while public_state.is_terminal == False:
        turn = public_state.turn
        action = players[turn].take_action()
        
        infos, public_state, person_states, private_state = env.forward(action)
        for i in range(len(players)):
            players[i].receive_info(infos[i])

   return public_state.scores                
</pre>



![the basic procedure of roomai](https://github.com/roomai/RoomAI/blob/master/roomai/game.png)

We define these basic concepts as classes in [roomai/common/common.py](https://github.com/roomai/RoomAI/blob/master/roomai/common/common.py), and all corresponding classes must extend them.  


#### 1. Information

Information is sent by the game environment to the players, and is the only way for the normal and chance players to access the states of the game environment. Info is consisted of the public state and the person state. 

1. The public state is available for all players.

2. A person state is corresponding to a player. The person state is available for the corresponding player and hidden from other players. 

3. Outside of Info, the private state is hidden from all players.

<pre>

class AbstractPrivateState:
    pass
    
class AbstractPublicState:
    turn             = 0
    ## players[turn] is expected to take an action
    ## for example, turn = 0 means the players[0] is expected to take an action
    ## default turn is 0

    action_history   = []
    ## The action_history records all actions taken by all players so far.
    ## For example, action_history = [(0, roomai.kuhn.KuhnAction.lookup(\"check\"),(1,roomai.kuhn.KuhnAction.lookup(\"bet\")]
    ## default action_history is []
    
    
    self.is_terminal = False
    self.scores      = None
    ## is_terminal = true means the game is over. At this time, scores is not None
    ## when is_terminal = true,  scores = [float0, float1, ..., float_n].
    ## when is_terminal = false, scores = None
    ## default is_terminal is False.



class AbstractPersonState:
    id                = None
    ## id = 0 means the player receiving this person state is players[0]

    available_actions = dict()
    ## If the corresponding player is expected to take a action,
    ## then available_actions is a dict with (action_key, action)
    ## Otherwise, available_actions is the empty dict.

class Info:
    public_state
    person_state
</pre>

The class Info sent to different players is different. It contains the same public state and different person states. The private_state isn't in any Info, hence no player can access it.

#### 2. Player

We implemented games in RoomAI in the [extensive form game](https://en.wikipedia.org/wiki/Extensive-form_game). The players in the extensive form game can be categorized into two types: the normal player and the 
chance player. The chance player is a kind of fictitious player, which generates the random events in the game. For example, the chance player
decides the hand cards of three players in DouDiZhu.

<pre>
class AbstractPlayer:
    def receive_info(self,info):
        raise NotImplementedError("The receiveInfo function hasn't been implemented") 

    def take_action(self):
        raise NotImplementedError("The takeAction function hasn't been implemented") 

    def reset(self):
        raise NotImplementedError("The reset function hasn't been implemented")
        
class AbstractChancePlayer:
    '''
    The chance player 
    '''
    def receive_info(self,info):
        raise NotImplementedError("The receiveInfo function hasn't been implemented") 

    def take_action(self):
        raise NotImplementedError("The takeAction function hasn't been implemented") 

    def reset(self):
        raise NotImplementedError("The reset function hasn't been implemented")
</pre>

In general, the chance player has the uniform distribution over the chance events. We have implemented the class RandomChancePlayer, which can be viewed as the default chance player.
<pre> 
class RandomChancePlayer(AbstractPlayer):
    '''
    The RandomChancePlayer is a chance player, who randomly takes an chance action.
    '''
    def receive_info(self, info):
        self.available_actions = info.person_state.available_actions

    def take_action(self):
        import random
        idx = int(random.random() * len(self.available_actions))
        return list(self.available_actions.values())[idx]

    def reset(self):
        pass
   
</pre>

To develop an AI-bot, you should extend this AbstractPlayer and implement the receive_info、take_action and reset function.



#### 3. Action

The player takes a action, and the game environment forwards with this action. The action in RoomAI
can be categorized in two two types: the normal action and the chance action. The chance action is for the chance players.

<pre>
class AbstractAction:
    @classmethod
    def lookup(self, key):
        raise NotImplementedError("The lookup function hasn't been implemented")

class AbstractChanceAction:
    @classmethod
    def lookup(self,key):
        raise NotImplementedError("The lookup function hasn't been implemented")

</pre>





#### 4. Enviroment

Enviroment is a environment of a game.

<pre>
class AbstractEnv:

    def backward(self):
        '''
        The game goes back to the previous states. 
        The backward function has been implemented in this abstract Env.
        To use the backward
        
        :return:infos, public_state, person_states, private_state 
        '''
        ....

    def init(self, params = dict()):
        '''
        Init the game environment
        
        :param: params for the game initialization. \n
        1. "num_normal_players" denotes how many normal players are in this game. \n
        2. "backward_enable" enables users to use the backward function. Default False. \n
        An example of params is {"num_normal_players":3, "backward_enable":True}
        :return: infos, public_state, person_states, private_state
        '''
        raise NotImplementedError("The init function hasn't been implemented")
        
    def forward(self, action):
        '''
        :return:infos, public_state, person_states, private_state 
        '''
        raise NotImplementedError("The forward function hasn't been implemented")


    #########  Some Utils Function
    @classmethod
    def compete(cls, env, players):
        '''
        holds a competition for the players, and computes the scores.
        '''
        raise NotImplementedError("The round function hasn't been implemented")

    @classmethod
    def available_actions(cls, public_state, person_state):
        '''
        :return all available_actions
        '''
        raise NotImplementedError("The available_actions function hasn't been implemented")

    @classmethod
    def is_action_valid(cls,action, public_state, person_state):
        raise NotImplementedError("The is_action_valid function hasn't been implemented")

</pre>




## Details of different games

If you want to develop an AI-bot for a particular game, you need to know the details of this game.
For example,  if you want to deveop an AI for TexasHoldem, you need to know where to find your hand cards.
You can find these information in the [API doc](http://roomai.readthedocs.io/en/latest/?badge=latest).

