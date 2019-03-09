## Object Relational Tutorial

Simple and complete applications may be constructed using just the ORM 
exclusively.


### Connecting

Using an in-memory SQLite database...  

```{python}
from sqlalchemy import create_engine
engine = create_engine('sqlite:///:memory:', echo=True
```

An instance of `Engine` represents the core interface to the
database, though with ORM we typically use this interface
behind the scenes.  

Note, `engine` above has not yet tried to connect to the
database yet, since that happens when the first task
against the database is requested.

### Declare a mapping

Start configuration by describing the db tables and by defining
how our classes will be mapped to those tables. In SQLAlchemy
this process uses a system called
[Declarative](
https://docs.sqlalchemy.org/en/latest/orm/extensions/declarative/index.html
). 

We'll usually have just one **declarative base class** per application.  

`users` is the table mapped to the class `User`. Any class using Declarative
requires `__tablename` and one primary key `Column`.

```{Python}
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    nickname = Column(String)

    def __repr__(self): # refactor to utilize fstrings?
       return "<User(name='%s', fullname='%s', nickname='%s')>" % (
                            self.name, self.fullname, self.nickname)
```

### Create a schema

See your `Table` object created by Declarative with `User.__table__`.
This object is a member of `MetaData`, a larger collection accessible
with the `.metadata` attribute of a `declarative_base` object.

Create the table metadata:
```{python}
Base.metadata.create_all(engine)
```
NOTE: this is simply a *minimal* table definition versus a verbose/full
definition. Check the docs for deets, if needed.


### Mapped class instance

Now we'll create a user and see what happens when we query a non-extant
`id` attribute.
```{Python}
ed_user = User(name = 'ed', fullname = 'Ed Jones', nickname = 'jonesy')
print(ed_user)
print(ed_user.name)
print(f"{str(None) if ed_user.id is None else None}")
```
Look mom, no `AttributeError`! :~)

### Session creation

Talk to the database using a `Session` class, a factory for `Session` 
objects.
```{Python}
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
# instantiate a new Session object to have a conversation
session = Session() # no connections to Engine until first transaction
```
Remember the connection pool is mainted by `Engine`.
### Add/update objects

`add()` the `User` object to our `Session`:
```{Python} 
session.add(ed_user) # instance is *pending*
```
The object is not yet represented by a row in the database; `Session` will
**flush**--issue necessary SQL commands to store the specific `User`
instance--as soon as `Ed Jones` is needed. 

To add more `User` objects we can do it batch-style w/`add_all()`:
```{python}
session.add_all([
    User(name='wendy', fullname='Wendy Webdings', nickname='ww'),
    User(name='mary', fullname='Mary Mack', nickname='the hammer'),
    User(name='fred', fullname='Fred Freitag', nickname='freezer'),
])
# check new pending changes
print(session.new)
# change ed's nickname
ed_user.nickname = 'eddie' # how 'bout the mob, eh?
# check changes
print(session.dirty)
# tell Session to issue changes and commit the transaction
session.commit() # flush the remaining changes to the db and commit
```
"Susequent operation with this session will occur in a **new** transaction,
which wil again re-acquire connection resources when first needed."
### Rollback 

We can rollback/revert changes within a transaction prior to a commit.
```{Python}
# make a shit change
ed_user.name = 'Edwardo'
# select it to see the change
print(session.query(User).filter(User.name.in_(['Edwardo'])).all())
# revert it
session.rollback()
assert len(session.query(User).filter(User.name.in_(['Edwardo'])).all()) is 0
```
### Querying


### Common filter operators

### Using other operators
