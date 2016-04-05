# ActiveRecordAssociations
A study of ActiveRecord's Associations and their generated methods.

####Questions: What methods are generated for each association type? What is the required database column in the matching table? How do the args passed to the `initialize` method affect the situation?

Experiment with three models. World, Character, PowerUp
 - World has many characters. Looking for `mushroom_kingdom.characters =>[...]`
 - Characters belong to a World. Looking for `mario.world => <#Mushroom_Kingdom Obj...>`
  
 - Character has many power ups. Looking for `mario.power_ups => [...]`
 - Power up has many characters. Looking for `fire_flower.characters => [...list of charcters that have this powerup]`
 
 - World has many powerups through characters. Looking for `mushroom_kingdom.power_ups => [...list of powerups from all characters]`
 - Power up belongs to world. Looking for `fire_flower.world => <#Mushroom_kingdom Obj>`

World:
```class World < ActiveRecord::Base
  has_many :characters
end```

Create and corresponding table with a column for the name of the world.
`rake db:create_migration NAME=create_world_table`




