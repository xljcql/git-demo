def sync_dianhan_mysql_loop(sleep_seconds=60):
    print(f"同步dianhan_mysql进程启动，每隔{sleep_seconds}秒检测一次...")
    while True:
        try:
            db = get_connection()
            cursor = db.cursor()
            # 获取当天日期字符串，格式和数据库中biaoDate一致，假设是 'YYYY-MM-DD'
            today_str = datetime.now().strftime('%Y-%m-%d')
            
            # 1. 查询当天所有dianhan记录
            cursor.execute("SELECT id, biaoDate, biaoTime FROM dianhan WHERE biaoDate = %s", (today_str,))
            rows = cursor.fetchall()
            
            for row in rows:
                id_, biaoDate, biaoTime = row
                ts_str = f"{biaoDate} {biaoTime}:00"
                cursor.execute("SELECT aValue FROM dian_mysql WHERE tS=%s", (ts_str,))
                result = cursor.fetchone()
                if result is not None:
                    aValue = result[0]
                    write_value = "非加碘" if (aValue is None or aValue == 0) else str(aValue)
                    cursor.execute("UPDATE dianhan SET dianhan_mysql=%s WHERE id=%s", (write_value, id_))
            
            db.commit()
            cursor.close()
            db.close()
        except Exception as e:
            print(f"{datetime.now()} 同步dianhan_mysql出错：", e)
        time.sleep(sleep_seconds)
