# Milestone Project: Query Performance Optimization

1. Retrieve all books authored by authors whose names start with 'J' and published after 2010.
    ```sql
    SELECT b.title, b.author_id, b.publication_date
    FROM books b
    JOIN authors a ON b.author_id = a.author_id
    WHERE a.author_name LIKE 'J%' AND b.publication_date > '2010-01-01';
    ```
    Answer :
    
    1. Penggunaan `WHERE` untuk `author_name` dapat dioptimize dengan memindahkan pencarian berdasarkan `author_name` ke `JOIN`. Hal ini dapat mengoptimize query karena pencarian berdasarkan `author_name` sudah difilter saat melakukan `JOIN`. Sedangkan ketika difilter pada kondisi `WHERE` maka data sudah dijoin terlebih dahulu sehingga tidak optimize. Berikut adalah query yang sudah dioptimize.

        ```sql
        SELECT b.title, b.author_id, b.publication_date
        FROM books b
        JOIN authors a ON b.author_id = a.author_id AND a.author_name LIKE 'J%'
        WHERE b.publication_date > '2010-01-01';
        ```

    2.  Kolom `author_name` pada tabel `authors` dan kolom `publication_date` pada tabel `books` dapat diberikan index karena memiliki tingkat keunikan yang tinggi. Berikut adalah query untuk membuat index.

        ```sql
        CREATE INDEX idx_authors_author_name ON authors (author_name);
        CREATE INDEX idx_books_publication_date ON books (publication_date);
        ```

2. Retrieve top 10 bestselling books by total sales quantity.
    ```sql
    SELECT b.title, SUM(s.quantity) AS total_sales
    FROM books b
    JOIN sales s ON b.book_id = s.book_id
    GROUP BY b.book_id
    ORDER BY total_sales DESC
    LIMIT 10;
    ```
    Answer :

    Dikarenakan tabel `sales` belum memiliki index untuk kolom `book_id`, maka ditambahkan index untuk kolom `book_id` pada tabel `sales`
    ```sql
    CREATE INDEX idx_sales_book_id ON sales (book_id);
    ```

3. Retrieve the average price of books in each genre.
    ```sql
    SELECT genre, AVG(price) AS avg_price
    FROM books
    GROUP BY genre;
    ```
    Answer :

    Menambahkan index untuk kolom `genre` pada tabel `books`.
    ```sql
    CREATE INDEX idx_books_genre ON books (genre);
    ```

4. Retrieve all sales transactions for a specific book.
    ```sql
    SELECT * FROM sales WHERE book_id = 456;
    ```
    Answer :

    - Penggunaan `*` bisa dioptimize dengan memilih kolom yang ingin didapatkan saja seperti `SELECT book_id, sale_date, quantity FROM sales WHERE book_id = 456;`
    - Index tidak perlu ditambahkan lagi karena sudah dibuat pada query no 2

5. Retrieve the total sales for each book published in the last year.
    ```sql
    SELECT b.title, SUM(s.quantity) AS total_sales
    FROM books b
    JOIN sales s ON b.book_id = s.book_id
    WHERE b.publication_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
    GROUP BY b.book_id;
    ```
    Answer :

    1. Penggunaan `WHERE` untuk `publication_date` dapat dioptimize dengan memindahkan pencarian berdasarkan `publication_date` ke `JOIN`. Hal ini dapat mengoptimize query karena pencarian berdasarkan `publication_date` sudah difilter saat melakukan `JOIN`. Sedangkan ketika difilter pada kondisi `WHERE` maka data sudah dijoin terlebih dahulu sehingga tidak optimize. Berikut adalah query yang sudah dioptimize.

        ```sql
        SELECT b.title, SUM(s.quantity) AS total_sales
        FROM books b
        JOIN sales s ON b.book_id = s.book_id 
            AND b.publication_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
        GROUP BY b.book_id;
        ```

    2.  Kolom `book_id` pada tabel `sales` dan kolom `publication_date` pada tabel `books` tidak perlu dibuatkan index karena sudah dibuat pada query no 1 dan 2.
