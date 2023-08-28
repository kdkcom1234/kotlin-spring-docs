Kotlin Exposed는 Kotlin에서 타입-세이프한 SQL을 작성할 수 있도록 도와주는 라이브러리입니다. 주문일자별 주문금액 합계를 처리하는 예제 코드는 아래와 같을 수 있습니다.

### 의존성 추가 (Gradle)
```gradle
dependencies {
    implementation 'org.jetbrains.exposed:exposed-core:0.34.1'
    implementation 'org.jetbrains.exposed:exposed-dao:0.34.1'
    implementation 'org.jetbrains.exposed:exposed-jdbc:0.34.1'
    implementation 'org.postgresql:postgresql:42.2.23'
}
```

### 데이터베이스 테이블 정의
```kotlin
import org.jetbrains.exposed.dao.IntIdTable
import org.jetbrains.exposed.sql.Column

object Orders : IntIdTable() {
    val date: Column<String> = varchar("date", 10)
    val amount: Column<Int> = integer("amount")
}
```

### 주문일자별 주문금액 합계 조회
```kotlin
import org.jetbrains.exposed.sql.Database
import org.jetbrains.exposed.sql.ResultRow
import org.jetbrains.exposed.sql.SqlExpressionBuilder.sum
import org.jetbrains.exposed.sql.selectAll
import org.jetbrains.exposed.sql.transactions.transaction

fun main() {
    // 데이터베이스 연결 설정
    Database.connect(
        "jdbc:postgresql://localhost:5432/mydatabase",
        driver = "org.postgresql.Driver",
        user = "myuser",
        password = "mypassword"
    )

    // 주문일자별 주문금액 합계 조회
    transaction {
        val result = Orders.slice(Orders.date, sum(Orders.amount))
            .selectAll()
            .groupBy(Orders.date)
            .orderBy(Orders.date)
            .map { rowToOrderSummary(it) }

        for (summary in result) {
            println("Date: ${summary.date}, Total Amount: ${summary.totalAmount}")
        }
    }
}

fun rowToOrderSummary(row: ResultRow): OrderSummary {
    return OrderSummary(
        date = row[Orders.date],
        totalAmount = row[sum(Orders.amount)] ?: 0
    )
}

data class OrderSummary(
    val date: String,
    val totalAmount: Int
)
```

이 코드는 `Orders` 테이블에서 주문일자(`date`)별로 주문금액(`amount`)을 합산하여 결과를 출력합니다. 결과는 주문일자로 정렬됩니다.

참고로, 실제 데이터베이스와 연동하려면 JDBC URL, 사용자 이름, 비밀번호 등을 적절히 설정해야 하며, `Orders` 테이블이 실제 데이터베이스에 존재해야 합니다.