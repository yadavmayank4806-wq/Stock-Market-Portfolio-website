#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
#define FILE_NAME "portfolio.txt"
#define INITIAL_CAPACITY 2
 
/* Ek stock ka record */
typedef struct {
    char  symbol[16];
    int   quantity;
    float buy_price;      /* average buy price */
    float current_price;
} Stock;
 
/* Poora portfolio: dynamic array of Stock */
typedef struct {
    Stock *stocks;        /* malloc/realloc se banega */
    int    count;
    int    capacity;
    float  realized_pl;   /* becne se hua profit/loss */
} Portfolio;
 
/* ---------- Memory management ---------- */
 
int initPortfolio(Portfolio *p) {
    p->stocks = (Stock *)malloc(INITIAL_CAPACITY * sizeof(Stock));
    if (p->stocks == NULL) {
        printf("Memory allocation failed!\n");
        return 0;
    }
    p->count = 0;
    p->capacity = INITIAL_CAPACITY;
    p->realized_pl = 0.0f;
    return 1;
}
 
void freePortfolio(Portfolio *p) {
    free(p->stocks);
    p->stocks = NULL;
    p->count = p->capacity = 0;
}
 
int growIfFull(Portfolio *p) {
    if (p->count < p->capacity)
        return 1;
    int newCap = p->capacity * 2;
    Stock *temp = (Stock *)realloc(p->stocks, newCap * sizeof(Stock));
    if (temp == NULL) {
        printf("Memory reallocation failed!\n");
        return 0;
    }
    p->stocks = temp;
    p->capacity = newCap;
    return 1;
}
 
/* ---------- Core operations ---------- */
 
/* Symbol dhundo, index return karo, nahi mila to -1 */
int findStock(const Portfolio *p, const char *symbol) {
    for (int i = 0; i < p->count; i++) {
        if (strcmp(p->stocks[i].symbol, symbol) == 0)
            return i;
    }
    return -1;
}
 
void buyStock(Portfolio *p, const char *symbol, int qty, float price) {
    if (qty <= 0 || price <= 0) {
        printf("Invalid quantity or price!\n");
        return;
    }
    int idx = findStock(p, symbol);
    if (idx != -1) {
        /* Pehle se hai: weighted average buy price nikalo */
        Stock *s = &p->stocks[idx];
        float totalCost = s->buy_price * s->quantity + price * qty;
        s->quantity += qty;
        s->buy_price = totalCost / s->quantity;
        s->current_price = price;
    } else {
        if (!growIfFull(p)) return;
        Stock *s = &p->stocks[p->count];
        strncpy(s->symbol, symbol, sizeof(s->symbol) - 1);
        s->symbol[sizeof(s->symbol) - 1] = '\0';
        s->quantity = qty;
        s->buy_price = price;
        s->current_price = price;
        p->count++;
    }
    printf("Bought %d share(s) of %s at %.2f\n", qty, symbol, price);
}
 
void sellStock(Portfolio *p, const char *symbol, int qty, float price) {
    int idx = findStock(p, symbol);
    if (idx == -1) {
        printf("Stock %s portfolio me nahi hai!\n", symbol);
        return;
    }
    Stock *s = &p->stocks[idx];
    if (qty <= 0 || price <= 0 || qty > s->quantity) {
        printf("Invalid quantity/price. Aapke paas sirf %d share(s) hain.\n", s->quantity);
        return;
    }
 
    float pl = (price - s->buy_price) * qty;
    p->realized_pl += pl;
    s->quantity -= qty;
    s->current_price = price;
 
    printf("Sold %d share(s) of %s at %.2f | %s: %.2f\n",
           qty, symbol, price, pl >= 0 ? "Profit" : "Loss", pl);
 
    /* Agar quantity 0 ho gayi to record hata do (shift left) */
    if (s->quantity == 0) {
        for (int i = idx; i < p->count - 1; i++)
            p->stocks[i] = p->stocks[i + 1];
        p->count--;
        printf("%s portfolio se remove ho gaya.\n", symbol);
    }
}
 
void updatePrice(Portfolio *p, const char *symbol, float newPrice) {
    int idx = findStock(p, symbol);
    if (idx == -1) {
        printf("Stock %s nahi mila!\n", symbol);
        return;
    }
    if (newPrice <= 0) {
        printf("Invalid price!\n");
        return;
    }
    p->stocks[idx].current_price = newPrice;
    printf("%s ka current price %.2f set ho gaya.\n", symbol, newPrice);
}
 
/* ---------- Calculations ---------- */
 
float stockProfitLoss(const Stock *s) {
    return (s->current_price - s->buy_price) * s->quantity;
}
 
float totalInvestment(const Portfolio *p) {
    float total = 0;
    for (int i = 0; i < p->count; i++)
        total += p->stocks[i].buy_price * p->stocks[i].quantity;
    return total;
}
 
float portfolioValue(const Portfolio *p) {
    float total = 0;
    for (int i = 0; i < p->count; i++)
        total += p->stocks[i].current_price * p->stocks[i].quantity;
    return total;
}
 
/* ---------- Display ---------- */
 
void viewHoldings(const Portfolio *p) {
    if (p->count == 0) {
        printf("\nPortfolio khaali hai.\n");
        return;
    }
    printf("\n%-4s %-10s %-8s %-10s %-10s %-12s %-10s\n",
           "No.", "Symbol", "Qty", "Buy", "Current", "P/L", "P/L %");
    printf("---------------------------------------------------------------------\n");
    for (int i = 0; i < p->count; i++) {
        const Stock *s = &p->stocks[i];
        float pl = stockProfitLoss(s);
        float pct = (s->buy_price > 0) ? (pl / (s->buy_price * s->quantity)) * 100 : 0;
        printf("%-4d %-10s %-8d %-10.2f %-10.2f %-12.2f %-9.2f%%\n",
               i + 1, s->symbol, s->quantity, s->buy_price, s->current_price, pl, pct);
    }
}
 
void showPerformance(const Portfolio *p) {
    float invested = totalInvestment(p);
    float value = portfolioValue(p);
    float unrealized = value - invested;
    float pct = (invested > 0) ? (unrealized / invested) * 100 : 0;
 
    printf("\n===== PORTFOLIO PERFORMANCE =====\n");
    printf("Total Investment   : %.2f\n", invested);
    printf("Current Value      : %.2f\n", value);
    printf("Unrealized P/L     : %.2f (%.2f%%)\n", unrealized, pct);
    printf("Realized P/L       : %.2f\n", p->realized_pl);
    printf("Net P/L (Total)    : %.2f\n", unrealized + p->realized_pl);
 
    if (p->count > 0) {
        int best = 0, worst = 0;
        for (int i = 1; i < p->count; i++) {
            if (stockProfitLoss(&p->stocks[i]) > stockProfitLoss(&p->stocks[best])) best = i;
            if (stockProfitLoss(&p->stocks[i]) < stockProfitLoss(&p->stocks[worst])) worst = i;
        }
        printf("Best Performer     : %s (%.2f)\n",
               p->stocks[best].symbol, stockProfitLoss(&p->stocks[best]));
        printf("Worst Performer    : %s (%.2f)\n",
               p->stocks[worst].symbol, stockProfitLoss(&p->stocks[worst]));
    }
}
 
/* ---------- File handling ---------- */
 
void saveToFile(const Portfolio *p) {
    FILE *fp = fopen(FILE_NAME, "w");
    if (fp == NULL) {
        printf("File open nahi ho payi!\n");
        return;
    }
    fprintf(fp, "%d %.2f\n", p->count, p->realized_pl);
    for (int i = 0; i < p->count; i++) {
        fprintf(fp, "%s %d %.2f %.2f\n", p->stocks[i].symbol, p->stocks[i].quantity,
                p->stocks[i].buy_price, p->stocks[i].current_price);
    }
    fclose(fp);
    printf("Portfolio '%s' me save ho gaya.\n", FILE_NAME);
}
 
void loadFromFile(Portfolio *p) {
    FILE *fp = fopen(FILE_NAME, "r");
    if (fp == NULL) {
        printf("Koi saved file nahi mili, naya portfolio start ho raha hai.\n");
        return;
    }
    int n;
    float realized;
    if (fscanf(fp, "%d %f", &n, &realized) != 2) {
        fclose(fp);
        printf("File format galat hai.\n");
        return;
    }
    p->realized_pl = realized;
    for (int i = 0; i < n; i++) {
        if (!growIfFull(p)) break;
        Stock *s = &p->stocks[p->count];
        if (fscanf(fp, "%15s %d %f %f", s->symbol, &s->quantity,
                   &s->buy_price, &s->current_price) != 4)
            break;
        p->count++;
    }
    fclose(fp);
    printf("%d stock(s) file se load hue.\n", p->count);
}
 
/* ---------- Main menu ---------- */
 
void printMenu(void) {
    printf("\n====== STOCK PORTFOLIO SIMULATOR ======\n");
    printf("1. Buy Stock\n");
    printf("2. Sell Stock\n");
    printf("3. View Holdings\n");
    printf("4. Update Current Price\n");
    printf("5. Portfolio Performance\n");
    printf("6. Save to File\n");
    printf("7. Exit\n");
    printf("Choice: ");
}
 
int main(void) {
    Portfolio portfolio;
    if (!initPortfolio(&portfolio))
        return 1;
 
    loadFromFile(&portfolio);
 
    int choice;
    char symbol[16];
    int qty;
    float price;
 
    do {
        printMenu();
        if (scanf("%d", &choice) != 1) {
            while (getchar() != '\n');   /* galat input clear karo */
            choice = 0;
        }
 
        switch (choice) {
            case 1:
                printf("Symbol: ");        scanf("%15s", symbol);
                printf("Quantity: ");      scanf("%d", &qty);
                printf("Buy price: ");     scanf("%f", &price);
                buyStock(&portfolio, symbol, qty, price);
                break;
            case 2:
                printf("Symbol: ");        scanf("%15s", symbol);
                printf("Quantity: ");      scanf("%d", &qty);
                printf("Sell price: ");    scanf("%f", &price);
                sellStock(&portfolio, symbol, qty, price);
                break;
            case 3:
                viewHoldings(&portfolio);
                break;
            case 4:
                printf("Symbol: ");        scanf("%15s", symbol);
                printf("New price: ");     scanf("%f", &price);
                updatePrice(&portfolio, symbol, price);
                break;
            case 5:
                showPerformance(&portfolio);
                break;
            case 6:
                saveToFile(&portfolio);
                break;
            case 7:
                saveToFile(&portfolio);
                printf("Exiting... Thank you!\n");
                break;
            default:
                printf("Invalid choice, dobara try karo.\n");
        }
    } while (choice != 7);
 
    freePortfolio(&portfolio);
    return 0;
}
 
