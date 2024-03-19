# Создание базы данных

>Ваша задача - создать веб-приложение, которое позволит пользователям обмениваться книгами между собой. Это приложение должно облегчать процесс обмена книгами, позволяя пользователям находить книги, которые им интересны, и находить новых пользователей для обмена книгами. Функционал веб-приложения должен включать следующее:
Создание профилей: Возможность пользователям создавать профили, указывать информацию о себе, своих навыках, опыте работы и предпочтениях по проектам.
Добавление книг в библиотеку: Пользователи могут добавлять книги, которыми они готовы поделиться, в свою виртуальную библиотеку на платформе.
Поиск и запросы на обмен: Функционал поиска книг в библиотеке других пользователей. Возможность отправлять запросы на обмен книгами другим пользователям.
Управление запросами и обменами: Возможность просмотра и управления запросами на обмен. Возможность подтверждения или отклонения запросов на обмен.

**Users - пользователи**

```

class Users(SQLModel, table=True):

    id: Optional[int] = Field(default=None, primary_key=True)
    username: str = Field(unique=True)
    hashed_password: str
    name: str
    surname: str
    email: str
    country: str
    city: str

    book_ownership: List["Books"] = Relationship(sa_relationship_kwargs={"cascade": "delete"}, back_populates="owner")

    reader: List["Books"] = Relationship(back_populates="in_readings", link_model=Readings)
    readings: List["Readings"] = Relationship(sa_relationship_kwargs={"cascade": "delete"}, back_populates="reader")

    reviews: List["Review"] = Relationship(back_populates="reviewer")

    __table_args__ = {"extend_existing": True}
    
```
**Books - книги**

```

class Books(SQLModel, table=True):

    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    author: str
    genre: Genres
    publisher: str
    year_of_publication: int
    description: str

    owner_id: Optional[int] = Field(default=None, foreign_key="users.id")
    owner: Optional[Users] = Relationship(back_populates="book_ownership")

    in_readings: List["Users"] = Relationship(back_populates="reader", link_model=Readings)
    book_read: List["Readings"] = Relationship(sa_relationship_kwargs={"cascade": "delete"}, back_populates="book")

    reviews: List["Review"] = Relationship(sa_relationship_kwargs={"cascade": "delete"}, back_populates="book")
    book_request: List["Requests"] = Relationship(back_populates="book")

    __table_args__ = {"extend_existing": True}
        
```
**Readings - вледение книгой на данный момент и статус владения**

```

class Readings(SQLModel, table=True):
    reader_id: Optional[int] = Field(default=None, foreign_key="users.id", primary_key=True)
    book_id: Optional[int] = Field(default=None, foreign_key="books.id", primary_key=True)
    status: ReadingStatus
    start_date: date = Field(default=date.today)
    end_date: date = Field(nullable=True)

    reader: Optional["Users"] = Relationship(back_populates="readings")
    book: Optional["Books"] = Relationship(back_populates="book_read")

```
**Requests - запросы на обмен**

```

class Requests(SQLModel, table=True):
    receiver_id: Optional[int] = Field(default=None, foreign_key="users.id", primary_key=True)
    sender_id: Optional[int] = Field(default=None, foreign_key="users.id", primary_key=True)
    book_id: Optional[int] = Field(default=None, foreign_key="books.id", primary_key=True)
    conditions: Optional[str]
    response: Optional[str] = None
    status: RequestStatus
    #sender: Optional[Users] = Relationship(back_populates="exchange_requests_sent", link_model=Users)
    sender: Optional[Users] = Relationship(sa_relationship_kwargs=dict(foreign_keys="[Requests.sender_id]"))
    #recipient: Optional[Users] = Relationship(back_populates="exchange_requests_received", link_model=Users)
    recipient: Optional[Users] = Relationship(sa_relationship_kwargs=dict(foreign_keys="[Requests.receiver_id]"))
    book: Optional[Books] = Relationship(back_populates="book_request")

        
```
**Reviews - отзывы на книги**

```

class Review(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    book_id: int = Field(foreign_key="books.id")
    reviewer_id: int = Field(foreign_key="users.id")
    rating: int  # Assuming a rating scale of 1-5
    comment: Optional[str] = None

    book: Optional[Books] = Relationship(back_populates="reviews")
    reviewer: Optional[Users] = Relationship(back_populates="reviews")

    __table_args__ = {"extend_existing": True}

        
```
**Жанры и статусы для разных таблиц**

```

class Genres(Enum):
    fantasy = "fantasy"
    detectives = "detectives"
    romance = "romance"
    thrillers = "thrillers"
    horrors = "horrors"
    comics = "comics"
    adventures = "adventures"
    poetry = "poetry"
    other = "other"


class ReadingStatus(Enum):
    exchanged = "exchanged"
    in_process = "in_process"
    available = "available"


class RequestStatus(Enum):
    sent = "sent"
    accepted = "accepted"
    rejected = "rejected"
    
```