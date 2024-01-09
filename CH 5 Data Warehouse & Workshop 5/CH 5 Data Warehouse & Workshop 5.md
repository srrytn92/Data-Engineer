# R2DE Workshop5
### Data Warehouse with BigQuery
&nbsp;<br>
&nbsp;<br>
#### สร้าง Dataset บน BigQuery เพื่อเตรียมโหลดข้อมูลเข้ามา
![create_dataset](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/9e113f76-0c2e-4957-ab60-55fa434b716f)

#### เข้าไปที่ Airflow ผ่าน Composer เพื่อสร้าง MySQL Connection เพื่อต่อกับ Database
![edit_connection](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/d03e9dc0-4399-41b0-ae95-69efdf8fade5)
#### เตรียม code python ที่สร้างไว้ เพื่อให้ Pipeline ทำงานอัตโนมัติ
<pre>
  <code class="language-python">
    from airflow.models import DAG
    from airflow.operators.bash import BashOperator
    from airflow.operators.python import PythonOperator
    from airflow.providers.mysql.hooks.mysql import MySqlHook
    from airflow.utils.dates import days_ago
    import pandas as pd
    import requests

    MYSQL_CONNECTION = "mysql_default"   # ชื่อของ connection ใน Airflow ที่เซ็ตเอาไว้
    CONVERSION_RATE_URL = "https://r2de2-workshop-vmftiryt6q-ts.a.run.app/usd_thb_conversion_rate"

    # path ที่จะใช้
    mysql_output_path = "/home/airflow/gcs/data/audible_data_merged1.csv"
    conversion_rate_output_path = "/home/airflow/gcs/data/conversion_rate1.csv"
    final_output_path = "/home/airflow/gcs/data/output1.csv"

    
    def get_data_from_mysql(transaction_path):
        # รับ transaction_path มาจาก task ที่เรียกใช้

        # เรียกใช้ MySqlHook เพื่อต่อไปยัง MySQL จาก connection ที่สร้างไว้ใน Airflow
        mysqlserver = MySqlHook(MYSQL_CONNECTION)
    
        # Query จาก database โดยใช้ Hook ที่สร้าง ผลลัพธ์ได้ pandas DataFrame
        audible_data = mysqlserver.get_pandas_df(sql="SELECT * FROM audible_data")
        audible_transaction = mysqlserver.get_pandas_df(sql="SELECT * FROM audible_transaction")

        # Merge data จาก 2 DataFrame เหมือนใน workshop1
        df = audible_transaction.merge(audible_data, how="left", left_on="book_id", right_on="Book_ID")

        # Save ไฟล์ CSV ไปที่ transaction_path ("/home/airflow/gcs/data/audible_data_merged.csv")
        # จะไปอยู่ที่ GCS โดยอัตโนมัติ
        df.to_csv(transaction_path, index=False)
        print(f"Output to {transaction_path}")

    def get_conversion_rate(conversion_rate_path):
        r = requests.get(CONVERSION_RATE_URL)
        result_conversion_rate = r.json()
        df = pd.DataFrame(result_conversion_rate)

        # เปลี่ยนจาก index ที่เป็น date ให้เป็น column ชื่อ date แทน แล้วเซฟไฟล์ CSV
        df = df.reset_index().rename(columns={"index": "date"})
        df.to_csv(conversion_rate_path, index=False)
        print(f"Output to {conversion_rate_path}")


    def merge_data(transaction_path, conversion_rate_path, output_path):
        # อ่านจากไฟล์ สังเกตว่าใช้ path จากที่รับ parameter มา
        transaction = pd.read_csv(transaction_path)
        conversion_rate = pd.read_csv(conversion_rate_path)

        transaction['date'] = transaction['timestamp']
        transaction['date'] = pd.to_datetime(transaction['date']).dt.date
        conversion_rate['date'] = pd.to_datetime(conversion_rate['date']).dt.date

        # merge 2 DataFrame
        final_df = transaction.merge(conversion_rate, how="left", left_on="date", right_on="date")
    
        # แปลงราคา โดยเอาเครื่องหมาย $ ออก และแปลงให้เป็น float
        final_df["Price"] = final_df.apply(lambda x: x["Price"].replace("$",""), axis=1)
        final_df["Price"] = final_df["Price"].astype(float)

        final_df["THBPrice"] = final_df["Price"] * final_df["conversion_rate"]
        final_df = final_df.drop(["date", "book_id"], axis=1)

        # save ไฟล์ CSV
        final_df.to_csv(output_path, index=False)
        print(f"Output to {output_path}")
        print("== End of Workshop 4 ʕ•́ᴥ•̀ʔっ♡ ==")


    with DAG(
        "workshop5_bq_load_dag_example",
        start_date=days_ago(1),
        schedule_interval="@once",
        tags=["workshop"]
    ) as dag:


        t1 = PythonOperator(
            task_id="get_data_from_mysql",
            python_callable=get_data_from_mysql,
            op_kwargs={
                "transaction_path": mysql_output_path,
            },
        )

        t2 = PythonOperator(
            task_id="get_conversion_rate",
            python_callable=get_conversion_rate,
            op_kwargs={
                "conversion_rate_path": conversion_rate_output_path,
            },
        )

        t3 = PythonOperator(
            task_id="merge_data",
            python_callable=merge_data,
            op_kwargs={
                "transaction_path": mysql_output_path,
                "conversion_rate_path": conversion_rate_output_path,
                "output_path" : final_output_path,
            },
        )

        t4 = BashOperator(
            task_id="bq_load",
            bash_command="bq load \
                --source_format=CSV \
                --autodetect \
                dataset.tablename \
                gs://GCS_Bucket/data/filename.csv"
        )

        [t1, t2] >> t3 >> t4
  </code>
</pre>

#### อัพโหลดไฟล์ Python ไปไว้ในโฟลเดอร์ dags
![upload_to_dag](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/21f5ef4f-55b8-4d1a-aa51-19e46dec7076)

#### เข้าไปดู Pipeline ที่สร้างไว้ใน Airflow
![bq_load](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/685b1404-033b-423b-a027-fb9251cf5e5e)

#### เมื่อ Pipeline ทำงานเสร็จแล้ว ข้อมูลก็จะเข้าไปอยู่ใน BigQuery
![table_big_query2](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/678e6567-71df-4ad0-881a-5545db4fe664)

