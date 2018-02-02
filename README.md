#include <iostream>
#include <cstdlib>
#include <fstream>
#include <chrono>
#include <cstring>
#include "gmp.h"
#include <cmath>

// szybkie potegowanie
long long power_modulo_fast(long long a, long long b, long long m) {
  long long result = 1;
  long long x = a % m;

  for (long long i = 1; i <= b; i <<= 1) {
    x %= m;
    if ((b & i) != 0) {
      result *= x;
      result %= m;
    }
    x *= x;
  }

  return result;
}

long long powerOf2[63] = {
    1 << 0,    1 << 1,    1 << 2,    1 << 3,    1 << 4,    1 << 5,
    1 << 6,    1 << 7,    1 << 8,    1 << 9,    1 << 10,   1 << 11,
    1 << 12,   1 << 13,   1 << 14,   1 << 15,   1 << 16,   1 << 17,
    1 << 18,   1 << 19,   1 << 20,   1 << 21,   1 << 22,   1 << 23,
    1 << 24,   1 << 25,   1 << 26,   1 << 27,   1 << 28,   1 << 29,
    1ll << 30, 1ll << 31, 1ll << 32, 1ll << 33, 1ll << 34, 1ll << 35,
    1ll << 36, 1ll << 37, 1ll << 38, 1ll << 39, 1ll << 40, 1ll << 41,
    1ll << 42, 1ll << 43, 1ll << 44, 1ll << 45, 1ll << 46, 1ll << 47,
    1ll << 48, 1ll << 49, 1ll << 50, 1ll << 51, 1ll << 52, 1ll << 53,
    1ll << 54, 1ll << 55, 1ll << 56, 1ll << 57, 1ll << 58, 1ll << 59,
    1ll << 60, 1ll << 61, 1ll << 62};

bool trialDivision(long long number) {
  // sprawdzanie czy liczba dzieli sie przez ktoras z nieparzystych liczb
  for (long long trial = 2, wall = sqrt(number) + 1; trial < wall; ++trial) {
    if (number % trial == 0) {  // jesli liczba number dzieli sie przez trial
      return false;             // zwroc false, bo znaleziono dzielnik liczby
    }
  }

  return true;  // zwroc true, bo nie znaleziono dzielnika liczby number
}

// metoda Millera-Rabina
bool millerRabin(long long n, int k) {
  long long s = 0;
  long long s2 = 1;
  long long a, d, i, r, prime;
  srand(time(NULL));

  if (n < 4) {
    return true;
  }

  if (n % 2 == 0) {
    return false;
  }

  // calculate s and d
  while ((s2 & (n - 1)) == 0) {
    s += 1;
    s2 <<= 1;
  }

  d = n / s2;

  // try k times
  for (i = 0; i < k; ++i) {
    a = 1 + (long long)((n - 1) * rand() / (RAND_MAX + 1.0));

    if (power_modulo_fast(a, d, n) != 1) {
      prime = 0;
      for (r = 0; r <= s - 1; ++r) {
        if (power_modulo_fast(a, powerOf2[r] * d, n) == n - 1) {
          prime = 1;
          break;
        }
      }

      if (prime == 0) {
        return false;
      }
    }
  }

  return true;
}

int main(int argc, char *argv[]) {
  const int reps = 30;  // liczba powtorzen
  const char *filename = "numbers.txt";
  std::fstream infile(filename);
  long long number;

  typedef std::chrono::high_resolution_clock Time;
  double trialDivisionTime = 0;
  double millerRabinTime = 0;
  double mpzTime = 0;
  bool isPrime;
  int result;

  mpz_t bignumber;
  std::string strnum;
  std::chrono::duration<double> time;

  while (!infile.eof()) {     // dopoki nie natrafiono na koniec pliku
    getline(infile, strnum);  // wczytanie liczby
    std::cout << "badana liczba: " << strnum;
    number = atoll(strnum.c_str());
    std::cout << "\n";

    // mierzenie czasu funkcji trialDivision
    std::cout << "\nMetoda Trial Division\n";
    auto start = Time::now();
    isPrime = trialDivision(number);
    auto stop = Time::now();
    time = stop - start;
    trialDivisionTime += time.count();
    std::cout << "czas dzialania: " << time.count() << "s\n";
    std::cout << "wynik: " << (isPrime ? "liczba jest pierwsza"
                                       : "liczba nie jest pierwsza");
    std::cout << "\n";

    // mierzenie czasu funkcji millerRabin
    std::cout << "\nMetoda Miller-Rabin\n";
    start = Time::now();
    isPrime = millerRabin(number, reps);
    stop = Time::now();
    time = stop - start;
    millerRabinTime += time.count();
    std::cout << "czas dzialania: " << time.count() << "s\n";
    std::cout << "wynik: " << (isPrime ? "liczba prawdopodobnie jest pierwsza"
                                       : "liczba nie jest pierwsza");
    std::cout << "\n";

    // mierzenie czasu funkcji mpz_probab_prime_p
    mpz_init(bignumber);
    mpz_set_str(bignumber, strnum.c_str(), 10);
    std::cout << "\nMetoda mpz_probab_prime_p\n";
    start = Time::now();
    result = mpz_probab_prime_p(bignumber, reps);
    stop = Time::now();
    time = stop - start;
    mpzTime += time.count();
    std::cout << "czas dzialania: " << time.count() << "s\n";
    std::cout << "wynik: ";
    switch (result) {
      case 0:
        std::cout << "liczba nie jest pierwsza\n";
        break;
      case 1:
        std::cout << "liczba prawdopodobnie jest pierwsza\n";
        break;
      case 2:
        std::cout << "liczba jest pierwsza\n";
        break;
    }
    mpz_clear(bignumber);

    std::cout << "\n\n";
  }

  std::cout << "\nCzas calkowity\n";
  std::cout << "Trial Division: " << trialDivisionTime;
  std::cout << "\nMiller-Rabin: " << millerRabinTime;
  std::cout << "\nmpz_probab_prime_p: " << mpzTime;
  std::cout << std::endl;

  system("PAUSE");
  return 0;
}
