# javalearning
Repository for different version java features and its samples


import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<BigDecimal> amounts = Arrays.asList(new BigDecimal("10"), new BigDecimal("20"), new BigDecimal("30"), new BigDecimal("40"), new BigDecimal("0.5"));
        BigDecimal total = new BigDecimal("100.5");

        Map<BigDecimal, BigDecimal> percentages = calculatePercentages(amounts, total);
        percentages.forEach((amount, percentage) -> System.out.println("Amount: " + amount + ", Percentage: " + percentage));
    }

    public static Map<BigDecimal, BigDecimal> calculatePercentages(List<BigDecimal> amounts, BigDecimal total) {
        Map<BigDecimal, BigDecimal> percentages = new LinkedHashMap<>();
        BigDecimal hundred = new BigDecimal("100");

        // Calculate the raw percentages and their floor values
        BigDecimal sumOfFloors = BigDecimal.ZERO;
        Map<BigDecimal, BigDecimal> floors = new HashMap<>();
        for (BigDecimal amount : amounts) {
            BigDecimal rawPercentage = amount.divide(total, 2, RoundingMode.HALF_UP).multiply(hundred);
            BigDecimal floor;
            if (rawPercentage.compareTo(new BigDecimal("0.5")) >= 0 && rawPercentage.compareTo(BigDecimal.ONE) < 0) {
                floor = BigDecimal.ONE;
            } else {
                floor = rawPercentage.setScale(0, RoundingMode.HALF_UP);
            }
            percentages.put(amount, rawPercentage);
            floors.put(amount, floor);
            sumOfFloors = sumOfFloors.add(floor);
        }

        // Calculate the remainders and sort by raw percentage
        BigDecimal remainderSum = hundred.subtract(sumOfFloors);
        List<Map.Entry<BigDecimal, BigDecimal>> rawPercentageList = new ArrayList<>(percentages.entrySet());
        rawPercentageList.sort(Map.Entry.comparingByValue());

        // Distribute the remaining percentages
        while (remainderSum.compareTo(BigDecimal.ZERO) > 0) {
            for (int i = rawPercentageList.size() - 1; i >= 0; i--) {
                Map.Entry<BigDecimal, BigDecimal> entry = rawPercentageList.get(i);
                BigDecimal amount = entry.getKey();
                BigDecimal floor = floors.get(amount);
                BigDecimal newPercentage = floor.add(BigDecimal.ONE);
                if (newPercentage.compareTo(hundred) <= 0) {
                    percentages.put(amount, newPercentage);
                    floors.put(amount, newPercentage);
                    remainderSum = remainderSum.subtract(BigDecimal.ONE);
                    if (remainderSum.compareTo(BigDecimal.ZERO) == 0) {
                        break;
                    }
                }
            }
        }

        // Replace raw percentages with their floor values
        for (Map.Entry<BigDecimal, BigDecimal> entry : percentages.entrySet()) {
            BigDecimal amount = entry.getKey();
            BigDecimal floor = floors.get(amount);
            percentages.put(amount, floor);
        }

        return percentages;
    }
}
