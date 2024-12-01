# 🌟 Meet People Database 🌟

By Francisco José Bretones López

📺 **Video overview**: [Watch here](https://youtu.be/N1yTOF-Avuk)

---

## 📜 Scope

- This project focuses on creating a database for the "Meet People" app.
- It allows users to connect with others who share similar interests.
- Users can form groups and organize parties to meet in person.

---

## 🗂️ Schema and Entities

### 👤 User's Table

- `id`: Integer. Unique identifier for each user. (Primary key)
- `username`: Text. Unique, not null. Identifies the username.
- `email`: Text. Not null. User's email address.
- `password`: Text. Hashed password for authentication purposes.
- `created_at`: Timestamp of user creation.

### 💡 Users_Interest Table

This table links users with their shared interests.

- `user_id`: Integer. Unique identifier for each user. (Primary key)
- `interest`: Text. Identifies user's interest.

- **Foreign Key**:
  - `user_id` references `User.id`.

### 👥 Groups Table

- `id`: Integer. Unique identifier for groups. (Primary key)
- `groupname`: Text. Name of the group.
- `description`: Text. Description of the group.
- `creator_id`: Integer. Identifies the user who created the group. (Foreign key references `users(id)`).
- `created_at`: Timestamp of group creation.

- **Constraints**:
  - Description can be empty but not null.
  - Only alphanumeric characters allowed in group names.

- **Foreign Key**:
  - `creator_id` references `users(id)`.

### 🔗 User_Groups Relation Table

A many-to-many relationship between Users and Groups.

- `user_id`: Integer. Not null. References the user who joined the group.
- `group_id`: Integer. Not null. References the group that the user joined.

- **Foreign Keys**:
  - `user_id` references `id` on `users` table.
  - `group_id` references `id` on `groups` table.

### 🎉 Parties Table

- `id`: Integer. Unique identifier for the party. (Primary key)
- `title`: Text. Name of the party.
- `start_date`: Date. Start date of the party.
- `end_date`: Date. End date of the party.
- `location`: Text. Location of the party.
- `group_id`: Integer. Not null. References the group associated with the party. (Foreign key referencing `groups(id)`).
- `creator_id`: Integer. Not null. User who created the party. (Foreign key referencing `users(id)`).

- **Constraint**:
  - `start_date` must be before `end_date`.

### 🔄 User_Party Relation Table

Represents a many-to-many relationship between Users and Parties.

- `party_id`: Integer. Not null. References the party being attended.
- `user_id`: Integer. Not null. References the user attending the party.
- `status`: Text. Not null. Indicates if the party is accepted, rejected, or pending approval. (Check: only allows 'pending', 'accepted', or 'rejected').

- **Foreign Keys**:
  - `party_id` references `parties(id)`.
  - `user_id` references `users(id)`.

### 💖 Users_matches Relation Table

Represents matches between two users.

- `id`: Integer. Not null. Identifies each match. (Primary key)
- `user1_id`, `user2_id`: Integer. Not null. Represents the two users involved in the match. They must always be different.
- `created_at`: Timestamp of match creation.
- `match_score`: Integer. The score of the match based on interests.

- **Foreign Keys**:
  - Both `user1_id` and `user2_id` reference `users(id)`.

### 💬 Comments Table

A section where users can post comments about parties.

- `id`: Integer. Not null. Identifies each comment. (Primary key)
- `text`: Text. Not null. Content of the comment.
- `rating`: Integer. Only values between 0 and 5.
- `created_at`: Timestamp of comment creation.
- `author_id`: Integer. Not null. Author of the comment.
- `party_id`: Integer. Not null. References the party being commented on.

- **Foreign Keys**:
  - `author_id` references `users(id)`.
  - `party_id` references `parties(id)`.

---

## 🗺️ Relationships

Below is an ER Diagram for the Meet People database:

![ER Diagram](https://i.imgur.com/61UzxRp.png)

---

## 🚀 Optimizations

### 📈 Indexes

To improve performance of searches:

- For `users` and `groups`:
```sql
CREATE INDEX user_groups_user_id_index ON users("id");
CREATE INDEX user_groups_group_id_index ON groups("id");

```

 * On `parties` table:

```sql
    CREATE INDEX user_party_user_idx ON users("id");
    CREATE INDEX user_party_parties_idx ON parties("id");
```

 * On `comments` table:

```sql
 CREATE INDEX "user_match1_idx" ON user_matches("user1_id");
CREATE INDEX "user_match2_idx" ON user_matches("user2_id");
```

- TRIGGERS:
    * `validate_party_dates`:
 ```sql
CREATE TRIGGER validate_party_dates
BEFORE UPDATE ON parties
FOR EACH ROW
WHEN NEW.start_date > NEW.end_date
BEGIN
    SELECT RAISE(ABORT, 'Start date must be before end date');
END;
```

 - This trigger is designed to ensure data integrity within the parties table by validating the relationship
    between the start and end dates of an event before an update operation.
    The primary purpose of this trigger is to prevent updates that would result in an event having a start date later than its end date. Such a condition would violate the logical consistency of the data.

    - Trigger Logic
        - Event Trigger: The trigger is set to execute BEFORE UPDATE operations on the parties table.

    - Condition Check: For each row being updated, the trigger checks if the new start date (NEW.start_date)
        is greater than the new end date (NEW.end_date).

    - Error Handling: If the condition is met (i.e., the start date is later than the end date), the trigger raises an error using RAISE(ABORT, 'Start date must be before end date').

    - Usage: To use this trigger effectively, ensure the following:
        · When updating an event in the parties table, the start date must always be earlier than or equal to the end date.
    - Example: -- Attempting to update an event with an invalid date range will trigger an error



- VIEWS:

    - `user_comments_count`: Show the number of comment for each user.
        The purpose of this view is to simplify the retrieval of information regarding the number of comments made by each user. It provides a convenient way to analyze and understand user comment activity.

```sql
CREATE VIEW user_comments_count AS
SELECT u.username AS user_username, u.id AS user_id, COUNT(c.id) AS comment_count
FROM users u
LEFT JOIN comments c ON u.id = c.author_id
GROUP BY u.id
ORDER BY comment_count DESC;
```


## Limitations

- While the database desing is generally well-structured and suitable for representing various aspects of the social platform "Meet People", there are certain scenarios and requirements for which it might not be the most optimal choice.
- Here are same of these limitations:

    * Limited `users` Interaction Tracking: The current desing captures users interactions through comments and parties attendance. If we needed more granular tracking it's possible that i would be necesary additional tables.

    * Limited `groups` Functionality: if we want to support more advanced funcionalities (e.g., file sharing) we might to expand the schema.

    * Limited `comments` Details : it would be necessary expand the schema if we want more datails about `comments`.
 
![CS50 Certificacion ](https://i.imgur.com/61UzxRp.png](https://github.com/FranBretones/CS50_SQL_Final_Project/blob/main/CS50%20SQL.pdf)
