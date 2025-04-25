# Catalog-1
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <math.h>

#define MAX_POINTS 20
#define MAX_LINE 512

typedef struct {
    int x;
    long long y;
} Point;

long long convertToDecimal(const char *value, int base) {
    long long result = 0;
    for (int i = 0; value[i]; i++) {
        char ch = tolower(value[i]);
        int digit = (isdigit(ch)) ? (ch - '0') : (ch - 'a' + 10);
        result = result * base + digit;
    }
    return result;
}

long long lagrangeInterpolation(Point points[], int k) {
    long long result = 0;

    for (int i = 0; i < k; i++) {
        double term = points[i].y;
        for (int j = 0; j < k; j++) {
            if (i != j) {
                term *= (0.0 - points[j].x) / (points[i].x - points[j].x);
            }
        }
        result += round(term);
    }
    return result;
}

void parseJSON(const char *filename, Point points[], int *k) {
    FILE *fp = fopen(filename, "r");
    if (!fp) {
        printf("Error opening file %s\n", filename);
        exit(1);
    }

    char line[MAX_LINE];
    int pointCount = 0;
    int reading = 0;

    while (fgets(line, sizeof(line), fp)) {
        if (strstr(line, "\"k\"")) {
            char *p = strchr(line, ':');
            *k = atoi(p + 1);
        }

        if (strchr(line, '{') && isdigit(line[1])) {
            int x = atoi(line);
            fgets(line, sizeof(line), fp); // base line
            int base = atoi(strchr(line, ':') + 2);

            fgets(line, sizeof(line), fp); // value line
            char *valStart = strchr(line, '\"') + 1;
            char *valEnd = strrchr(line, '\"');
            *valEnd = '\0';

            long long y = convertToDecimal(valStart, base);

            points[pointCount].x = x;
            points[pointCount].y = y;
            pointCount++;
        }
    }

    fclose(fp);
}

int main() {
    const char *files[] = { "testcase1.json", "testcase2.json" };
    int total = 2;

    for (int i = 0; i < total; i++) {
        Point points[MAX_POINTS];
        int k = 0;

        parseJSON(files[i], points, &k);

        long long secret = lagrangeInterpolation(points, k);
        printf("Secret (c) from %s: %lld\n", files[i], secret);
    }

    return 0;
}
 
