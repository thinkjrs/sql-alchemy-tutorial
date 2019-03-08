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

### Mapped class instance

### Session creation

### Add/update objects

### Rollback 

### Querying

### Common filter operators

### Using other operators
