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
```ruby
class World < ActiveRecord::Base
  has_many :characters
end
```

Create and corresponding table with a column for the name of the world.
`rake db:create_migration NAME=create_worlds_table`

Note the plural version of the class is used for the table.
```ruby
class CreateWorldTable < ActiveRecord::Migration
  def change
    create_table :worlds do |t|
      t.string :name
    end
  end
end
```
`rake db:migrate`
`db/schema.rb` now has this:

```ruby
ActiveRecord::Schema.define(version: 20160405222201) do

  create_table "worlds", force: :cascade do |t|
    t.string "name"
  end

end
```
Fire up `tux`.

Let's create a world that has no name. It is a simple instance, with no attributes.
```ruby
>> nameless_world = World.create
D, [2016-04-05T22:33:05.639088 #3388] DEBUG -- :    (0.2ms)  begin transaction
D, [2016-04-05T22:33:05.644635 #3388] DEBUG -- :   SQL (0.5ms)  INSERT INTO "worlds" DEFAULT VALUES
D, [2016-04-05T22:33:05.657346 #3388] DEBUG -- :    (12.0ms)  commit transaction
=> #<World id: 1, name: nil>
```
```ruby
nameless_world_methods = nameless_world.methods.map {|method| method.to_s}.sort!
nameless_world_methods.each {|m| puts m}
```
###Instance Methods for World
```ruby
#List of world instance methods - list of class Empty< ActiveRecord::Base

after_add_for_characters
after_add_for_characters=
after_add_for_characters?
after_remove_for_characters
after_remove_for_characters=
after_remove_for_characters?
autosave_associated_records_for_characters
before_add_for_characters
before_add_for_characters=
before_add_for_characters?
before_remove_for_characters
before_remove_for_characters=
before_remove_for_characters?
character_ids
character_ids=
characters
characters=
validate_associated_records_for_characters
```
###Class Methods for World
```ruby
after_add_for_characters
after_add_for_characters?
after_add_for_characters=

after_remove_for_characters
after_remove_for_characters?
after_remove_for_characters=

before_add_for_characters
before_add_for_characters?
before_add_for_characters=

before_remove_for_characters
before_remove_for_characters?
before_remove_for_characters=
```
### Character belongs to World
If a model is the child of something it gets a `belongs_to` association. If a model has two or parents it can belong to both through mutltiple `belongs_to` method calls. Another way of looking at the belongs to association is to ask youself if you want this kind of method: `child.dad => <#Dad Object>` or `child.mom => <#Mom Object`

If a model is a child of anything, it requires a parent_id column to be added to it's corresponding table. Any column with a `_id` suffix becomes a foreign key that points to the corresponding table's id column. So, if a character is a child of world, it will need a world_id column to point to the worlds table's id column.
```ruby
class Character < ActiveRecord::Base
  belongs_to :world
end
```
`rake db:create_migration NAME=create_characters_table`
```ruby
class CreateCharactersTable < ActiveRecord::Migration
  def change
    create_table :characters do |t|
      t.string :name
      t.integer :world_id
    end
  end
end
```
`rake db:migrate`
`db/schema.rb` now looks like this:
```ruby
ActiveRecord::Schema.define(version: 20160405235235) do

  create_table "characters", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "worlds", force: :cascade do |t|
    t.string "name"
  end

end
```

Let's see what methods our Character Class/instance has. Time to create a nameless character.
`tux`
```ruby
>> nameless_character = Character.new
=> #<Character id: nil, name: nil, world_id: nil>
>> nameless_character_methods = nameless_character.methods.map {|method| method.to_s}.sort!
>> nameless_character_methods.each {|m| puts m}
```
###Character instance methods.
```ruby
(nameless_character_methods - nameless_world_methods).each {|m| puts m}                                                                                                        
autosave_associated_records_for_world
belongs_to_counter_cache_after_update
build_world
create_world
create_world!
reset_world_id!
restore_world_id!
world
world=
world_id
world_id=
world_id?
world_id_before_type_cast
world_id_came_from_user?
world_id_change
world_id_changed?
world_id_was
world_id_will_change!
```

###Character Class methods
```ruby
>> (Character.methods - World.methods).each {|m| puts m}                                                     
=> []
```
