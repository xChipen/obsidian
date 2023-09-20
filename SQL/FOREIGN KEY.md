[[SQL]]
create table XX1(
ID varchar(10) primary key,
name varchar(10)
)

create table XX2(
ID varchar(10) primary key,
XX1ID varchar(10),
name varchar(10),
FOREIGN KEY (XX1ID) REFERENCES XX1(ID) ON DELETE CASCADE
)