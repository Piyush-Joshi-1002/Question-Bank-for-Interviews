Question no. 1. 
In the Amazon maintenance center, there exist n machines, each equipped with m (m ≥ 2) power units. The power of the f power unit in the th machine is denoted as machine_powers(iJi]. The strength of a machine is defined as the minimum power among all its power units.
We want to maximise the sum of strengths of all the machines. For this, we can perform the following 3 step operation multiple times (possibly 0).
• Select a machine that has not yet been marked.
• Transfer any one power unit from the selected machine to any other machine.
• Mark the selected machine and it cannot be chosen for further operations. However, marked machines can still receive power units from other machines.
Find the maximum sum of strengths of all machines.
Note:
• Each machine can transfer at most one power unit during the entire process, only when it is unmarked.
• A machine can receive power units from multiple other machines, even after it has been marked.
• The operation can be performed multiple times, but only unmarked machines can be selected during each operation.

<img width="301" alt="Screenshot 2025-06-12 at 3 12 18 PM" src="https://github.com/user-attachments/assets/9d368d30-d6a1-4d42-89a4-e428945d0e02" />

<img width="301" alt="Screenshot 2025-06-12 at 3 12 18 PM" src="https://github.com/user-attachments/assets/32179905-fd5a-42a2-908e-32818d4eb45f">

Code : 
```java
import java.util.*;

public class Main {
    public static long getMaximumStrengthSum(List<List<Integer>> machinePowers) {
        int n = machinePowers.size();
        int m = machinePowers.get(0).size();

        // Sort each machine's power units
        List<List<Integer>> sortedMachines = new ArrayList<>();
        for (List<Integer> machine : machinePowers) {
            List<Integer> sorted = new ArrayList<>(machine);
            Collections.sort(sorted);
            sortedMachines.add(sorted);
        }

        // Collect smallest and next smallest units
        List<Integer> smallestUnits = new ArrayList<>();
        List<Integer> nextSmallest = new ArrayList<>();
        for (List<Integer> machine : sortedMachines) {
            smallestUnits.add(machine.get(0));
            nextSmallest.add(machine.get(1));
        }

        // Calculate total of smallest units
        long totalTransferred = 0;
        for (int unit : smallestUnits) {
            totalTransferred += unit;
        }

        // Find the global smallest unit
        int globalSmallest = Collections.min(smallestUnits);

        long maxSum = 0;
        // Precompute the sum of nextSmallest units
        long sumNextSmallest = 0;
        for (int unit : nextSmallest) {
            sumNextSmallest += unit;
        }

        // Iterate to find the best machine to receive all transfers
        for (int i = 0; i < n; i++) {
            long transferred = totalTransferred - smallestUnits.get(i);
            int newStrength = Math.min(sortedMachines.get(i).get(0), globalSmallest);
            long sum = sumNextSmallest - nextSmallest.get(i) + newStrength;
            if (sum > maxSum) {
                maxSum = sum;
            }
        }
        return maxSum;
    }

    public static void main(String[] args) {
        List<List<Integer>> machinePowers1 = new ArrayList<>();
        machinePowers1.add(Arrays.asList(1, 5));
        machinePowers1.add(Arrays.asList(4, 3));
        machinePowers1.add(Arrays.asList(2, 10));
        System.out.println(getMaximumStrengthSum(machinePowers1)); // Expected Output: 9

        List<List<Integer>> machinePowers2 = new ArrayList<>();
        machinePowers2.add(Arrays.asList(2, 7, 4));
        machinePowers2.add(Arrays.asList(2, 4, 3));
        System.out.println(getMaximumStrengthSum(machinePowers2)); // Expected Output: 5
    }
}
```


Question : 2 

The developers at Amazon IAM are working on identifying vulnerabilities in their key generation process. The key is represented by an array of n integers, where the fh integer is denoted by keyli]. The vulnerability factor of the array (key) is defined as the maximum length of a subarray that has a Greatest Common Divisor (GCD) greater than 1.
You are allowed to make a maximum of maxChange modifications to the array, where each modification consists of changing any one element in the array to any other value.
Given an integer array key and an integer maxchange, find the least possible vulnerability factor of the key after making at most maxchange changes.
Note: The length of an empty subarray is considered to be zero.
Example
key = [2, 2, 4, 9, 6)
maxChange = 1
The length of key, n = 5. Here are some possible
changes to make:
1. Change the first element to 3. The array becomes
key = [3, 2, 4, 9, 6). The length of the longest
subarray with a GCD greater than 1 is 2 for the subarrays (2, 4] and [9, 6).
2. Change the third element of the array to 5. The
array becomes key = [2, 2, 5, 9, 6]. In this case, the
length of the longest subarray with a GCD greater than 1 is 2 for the subarrays [2, 2] and [9, 6].

<img width="289" alt="Screenshot 2025-06-12 at 3 18 34 PM" src="https://github.com/user-attachments/assets/34cdeb39-9b39-4e25-a384-86090c329a09" />
<br />

<img width="292" alt="Screenshot 2025-06-12 at 3 17 25 PM" src="https://github.com/user-attachments/assets/df29bd8a-6fcb-4a39-bbe7-46d0c09a2871" />
<br />
<img width="298" alt="Screenshot 2025-06-12 at 3 17 58 PM" src="https://github.com/user-attachments/assets/c5ece1be-467b-4629-93f0-629f38eda7db" />
<br />

```java
import java.util.Arrays;
import java.util.List;
import java.util.ArrayList; // Added for convenience in main for list creation

public class KeyVulnerability {

    private static int gcd(int a, int b) {
        while (b != 0) {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return a;
    }

    private int[][] st;
    private int[] logTable;
    private int n;

    private void buildSparseTable(List<Integer> arr) {
        this.n = arr.size();
        int maxLogN = (int) (Math.log(n) / Math.log(2));
        st = new int[maxLogN + 1][n];
        logTable = new int[n + 1];

        logTable[1] = 0;
        for (int i = 2; i <= n; i++) {
            logTable[i] = logTable[i / 2] + 1;
        }

        for (int i = 0; i < n; i++) {
            st[0][i] = arr.get(i);
        }

        for (int k = 1; k <= maxLogN; k++) {
            for (int i = 0; i + (1 << k) <= n; i++) {
                st[k][i] = gcd(st[k - 1][i], st[k - 1][i + (1 << (k - 1))]);
            }
        }
    }

    private int queryGcd(int L, int R) {
        if (L > R) {
            return 0;
        }
        int k = logTable[R - L + 1];
        return gcd(st[k][L], st[k][R - (1 << k) + 1]);
    }

    private boolean check(int targetK, List<Integer> key, int maxChange) {
        if (targetK == 0) {
            int changesNeeded = 0;
            for (int x : key) {
                if (x != 1) {
                    changesNeeded++;
                }
            }
            return changesNeeded <= maxChange;
        }

        int modificationsNeeded = 0;
        int lastModifiedIndex = -1;

        for (int i = 0; i <= n - (targetK + 1); i++) {
            if (i <= lastModifiedIndex) {
                continue;
            }

            int currentSegmentEnd = i + targetK;
            int currentGcd = queryGcd(i, currentSegmentEnd);

            if (currentGcd > 1) {
                modificationsNeeded++;
                lastModifiedIndex = currentSegmentEnd;
            }

            if (modificationsNeeded > maxChange) {
                return false;
            }
        }
        return true;
    }

    public int findLeastVulnerabilityFactor(List<Integer> key, int maxChange) {
        this.n = key.size();
        if (n == 0) {
            return 0;
        }

        buildSparseTable(key);

        int low = 0;
        int high = n;
        int ans = n;

        while (low <= high) {
            int mid = low + (high - low) / 2;
            if (check(mid, key, maxChange)) {
                ans = mid;
                high = mid - 1;
            } else {
                low = mid + 1;
            }
        }
        return ans;
    }

    public static void main(String[] args) {
        KeyVulnerability solver = new KeyVulnerability();

        List<Integer> key1 = Arrays.asList(2, 2, 4, 9, 6);
        int maxChange1 = 1;
        System.out.println("Key: " + key1 + ", maxChange: " + maxChange1 +
                           " -> Least Vulnerability Factor: " + solver.findLeastVulnerabilityFactor(key1, maxChange1));

        List<Integer> key2 = Arrays.asList(7, 14, 21, 28);
        int maxChange2 = 0;
        System.out.println("Key: " + key2 + ", maxChange: " + maxChange2 +
                           " -> Least Vulnerability Factor: " + solver.findLeastVulnerabilityFactor(key2, maxChange2));

        List<Integer> key3 = Arrays.asList(5, 10, 15, 20);
        int maxChange3 = 1;
        System.out.println("Key: " + key3 + ", maxChange: " + maxChange3 +
                           " -> Least Vulnerability Factor: " + solver.findLeastVulnerabilityFactor(key3, maxChange3));

        List<Integer> key4 = Arrays.asList(1, 1, 1, 1, 1);
        int maxChange4 = 0;
        System.out.println("Key: " + key4 + ", maxChange: " + maxChange4 +
                           " -> Least Vulnerability Factor: " + solver.findLeastVulnerabilityFactor(key4, maxChange4));

        List<Integer> key5 = Arrays.asList(2, 3, 5, 7, 11);
        int maxChange5 = 0;
        System.out.println("Key: " + key5 + ", maxChange: " + maxChange5 +
                           " -> Least Vulnerability Factor: " + solver.findLeastVulnerabilityFactor(key5, maxChange5));
    }
}

```

