## ActiveRecord Associations
A study of ActiveRecord's Associations and their generated methods.

#####Questions: What methods are generated for each association type? What is the required database column in the matching table? How do the args passed to the `initialize` method affect the situation?

We'll experiment with four models: Game, World, Character, and PowerUp.

A World `has_many` characters and powerups. We're looking for the following behaviour:
 - `mushroom_kingdom.characters =>[...]`
 - `mushroom_kingdom.power_ups => [...]`
 
Characters and Powerups both `belong_to` a World.
 - `mario.world => #<Mushroom Kingdom...>`
 - `fire_flower`.world => #<Mushroom Kingdom...>

A Character `has_many` powerups available `through` the world. 
 - `mario.power_ups => [...]`

A Powerup `belongs_to` _many_ characters. 
 - `fire_flower.characters => [...]`

Additionally, a Game has many characters but a character also has many games.
 - `mario.games => [...]`
 - `game.characters => [...]`
 
##Control
First we need a control model to compare our results against. This will be a blank class that inherits from ActiveRecord. The only column it will have will be its automatically generated `id` column.
```ruby
class Blank < ActiveRecord::Base
end

class CreateBlanksTable < ActiveRecord::Migration
  def change
    create_table :blanks do |t|
    end
  end
end
```

```ruby
control = Blank.new
=> #<Blank id: nil>
control_methods = control.methods.map {|m| m.to_s}.sort!
control_class_methods = Blank.methods.map {|m| m.to_s}.sort!
```
The list of control methods can be found [here](https://github.com/MooseBoost/ActiveRecordAssociations/blob/master/control_instance_methods.md "Instance Methods") and [here](https://github.com/MooseBoost/ActiveRecordAssociations/blob/master/control_class_methods.md "Class Methods").

##World
```ruby
class World < ActiveRecord::Base
  has_many :characters
end
```

Create a corresponding table with a column for the name of the world.
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

Migrate the change over to the database.
```ruby
rake db:migrate
```

`db/schema.rb` now has the following:
```ruby
ActiveRecord::Schema.define(version: 20160405222201) do

  create_table "worlds", force: :cascade do |t|
    t.string "name"
  end

end
```

Let's now create a world that has no name. It will be a simple instance, with no attributes.
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

We are now ready to make our comparisons.



###A _World_ that `has_many` _characters_
Note: The list of methods was created like this:
```ruby
>> nameless_world_methods = nameless_world.methods.map {|m| m.to_s}.sort!
>> (nameless_world_methods - control_methods).each {|m| puts m}

>> world_class_methods = World.methods.map {|m| m.to_s}.sort!
>> (world_class_methods - control_class_methods).each {|m| puts m}
```

Again, here is our model and its table.
```ruby
class World < ActiveRecord::Base
  has_many :characters
end
```
```ruby
class CreateWorldTable < ActiveRecord::Migration
  def change
    create_table :worlds do |t|
      t.string :name
    end
  end
end
```

#####Instance Methods
Generated from the `has_many` method
```ruby
after_add_for_characters
after_add_for_characters=
after_add_for_characters?
after_remove_for_characters
after_remove_for_characters=
after_remove_for_characters?

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

autosave_associated_records_for_characters
validate_associated_records_for_characters
```

Generated from the database columns:
```ruby
name
name=
name?
name_before_type_cast
name_came_from_user?
name_change
name_changed?
name_was
name_will_change!
reset_name!
restore_name!
```

#####Class Methods
```ruby
after_add_for_characters
after_add_for_characters=
after_add_for_characters?
after_remove_for_characters
after_remove_for_characters=
after_remove_for_characters?

before_add_for_characters
before_add_for_characters=
before_add_for_characters?
before_remove_for_characters
before_remove_for_characters=
before_remove_for_characters?
```

As you can see, ActiveRecord generates instance methods based on both names of the columns in the database and the various association macros.



###A _Character_ `belongs_to` a _world_
If a model is a child of something it gets a `belongs_to` association. If a model has two or parents it can belong to both through mutltiple `belongs_to` method calls. Another way of looking at the belongs to association is to ask youself if you want this kind of method: `child.dad => <#Dad Object>` or `child.mom => <#Mom Object`

If a model is a child of anything, it requires a parent_id column to be added to it's corresponding table. Any column with a `_id` suffix becomes a foreign key that points to the corresponding table's id column. So, if a character is a child of world, it will need a world_id column to point to the worlds table's id column.

Here is our Character model.
```ruby
class Character < ActiveRecord::Base
  belongs_to :world
end
```
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

Here is a nameless character.
```ruby
>> nameless_character = Character.new
=> #<Character id: nil, name: nil, world_id: nil>
>> nameless_character_methods = nameless_character.methods.map {|method| method.to_s}.sort!
>> (nameless_character_methods - control_methods).each {|m| puts m}
```
###Instance methods.
Generated from the `belongs_to` method:
```ruby
world
world=
autosave_associated_records_for_world
belongs_to_counter_cache_after_update
build_world
create_world
create_world!
```
Generated from the database columns:
```ruby
name
name=
name?
name_before_type_cast
name_came_from_user?
name_change
name_changed?
name_was
name_will_change!
reset_name!
restore_name!

world_id
world_id=
world_id?
world_id_before_type_cast
world_id_came_from_user?
world_id_change
world_id_changed?
world_id_was
world_id_will_change!
reset_world_id!
restore_world_id!
```

###Class methods
```ruby
>> (character_class_methods - control_class_methods).each {|m| puts m}
=> []
```


###A _Character_ `has_many` _power_ups_ `through:` _world_ 
At this point, we've looked at the generated methods we end up with when our class has both a `belongs_to` relationship and a `has_many` relationship. A PowerUp model with both of those relationships would have both sets of methods, along with the database-column generated methods. But what about this _many to many_ relationship, AKA a `has_many through:` association, that we've got going on between our Characters and their many powerups? A Character can have many powerups, like this: `nameless_character.powerups => [...]` but we also want a list of every character that has each powerup, like this: `nameless_powerup.characters => [...]`.

Here are updated versions of each of our three classes.
```ruby
class World < ActiveRecord::Base
  has_many :characters
  has_many :power_ups
end
```
```ruby
class Character < ActiveRecord::Base
  belongs_to :world
  has_many :power_ups, through: :world
end
```
```ruby
class PowerUp < ActiveRecord::Base
  belongs_to :world
  has_many :characters
end
```
And here is our current database schema.
Note: The name of the `power-ups` table has an underscore where the class name had an uppercase letter.
```ruby
ActiveRecord::Schema.define(version: 20160406024221) do

  create_table "characters", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "power_ups", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "worlds", force: :cascade do |t|
    t.string "name"
  end

end
```
If we look back to our generated methods, we see that it is the `has_many` macro that gives our ____s method. Our PowerUp has many characters and our characters will have many powerups through their world.

###It'sa Me....
I've gone ahead and created the world `mushroom_kingdom` with a name of "Mushroom Kingdom". I've also created two characters named Mario and Luigi along with two Powerups `mushroom`, and `fire_flower`.

Before anything else, lets test some of our relationships. 
If we give Mario a world to live in, like this:
```ruby
mario.world = mushroom_kingdom
```
we expect our `mushroom_kingdom.characters` to return some info about its single inhabitant, Mario.
Instead, however, we get this:
```ruby
>> mushroom_kingdom.characters.first
=> nil
```
Hmmm. The problem is this: 
**Children are not responsible**.
Rather than telling a child object (`mario`)something the parent needs to know, always tell the parent (`mushroom_kingdom`). The parent is the responsible one in the relationship and will let the child object know what it needs to know.
```ruby
>> goomba = Character.create(name: "Goomba")
=> #<Character id: 3, name: "Goomba", world_id: nil>

>> mushroom_kingdom.characters << goomba
>> goomba.world
=> #<World id: 1, name: "Mushroom Kingdom">
>> goomba
=> #<Character id: 3, name: "Goomba", world_id: 1>

>> mushroom_kingdom.power_ups << mushroom
>> mushroom_kingdom.power_ups << fire_flower

>> fire_flower.world
=> #<World id: 1, name: "Mushroom Kingdom">
```

So our associations are starting to come together - as long as we tell the parent. So how about the association between our characters and our powerups?
```ruby
>> mario.power_ups.first.name
=> "Mushroom"
```
Seems to work. What about asking our fire flower which characters have access to it?
```ruby
>>fire_flower.characters.first.name
=> ActiveRecord::StatementInvalid: SQLite3::SQLException: no such column: characters.power_up_id:
```
So this is looking for a `power_up_id` column. Recall that any column ending in `_id` is a foreign key column that goes hand in hand with a `belongs_to` method for a parent class. So this is essentially saying that our PowerUp model needs to be the parent of our characters. The problem is, is that we don't want that. We want our powerups to get their list of characters through whatever characters happen to live in the same world they do, not because they are directly related to them.




