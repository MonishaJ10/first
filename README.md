CsvMergeService.java (Service Layer)
java
Copy
Edit
package com.example.csvmerger.service;

import org.apache.commons.csv.*;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.util.*;

@Service
public class CsvMergeService {

    public File mergeCsvFiles(MultipartFile initialFile, MultipartFile kondorFile) throws IOException {
        File mergedFile = File.createTempFile("merged_output", ".csv");

        Reader initialReader = new InputStreamReader(initialFile.getInputStream());
        Reader kondorReader = new InputStreamReader(kondorFile.getInputStream());

        Iterable<CSVRecord> initialRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(initialReader);
        Iterable<CSVRecord> kondorRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(kondorReader);

        Set<String> headers = new LinkedHashSet<>();
        headers.addAll(initialRecords.iterator().next().toMap().keySet());
        headers.addAll(kondorRecords.iterator().next().toMap().keySet());
        headers.add("Type Nature");

        if (!headers.contains("Base Currency")) {
            headers.add("Base Currency");
        }

        FileWriter out = new FileWriter(mergedFile);
        CSVPrinter printer = new CSVPrinter(out, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])));

        // Add initial margin rows
        for (CSVRecord record : initialRecords) {
            Map<String, String> row = new HashMap<>(record.toMap());
            row.put("Type Nature", "Type Nature-OP");  // Add default
            printer.printRecord(getRow(headers, row));
        }

        // Add kondor rows
        for (CSVRecord record : kondorRecords) {
            Map<String, String> row = new HashMap<>(record.toMap());
            row.put("Type Nature", record.get("Type Nature"));

            // Base Currency from column index 21 (column 22)
            if (!row.containsKey("Base Currency") && record.size() > 21) {
                row.put("Base Currency", record.get(21));
            }

            printer.printRecord(getRow(headers, row));
        }

        printer.flush();
        printer.close();
        return mergedFile;
    }

    public void mergeCsvFilesFromPaths(String initialPath, String kondorPath, String outputPath) throws IOException {
        mergeCsvFiles(
                new org.springframework.mock.web.MockMultipartFile("initial", new FileInputStream(initialPath)),
                new org.springframework.mock.web.MockMultipartFile("kondor", new FileInputStream(kondorPath))
        ).renameTo(new File(outputPath));
    }

    private List<String> getRow(Set<String> headers, Map<String, String> row) {
        List<String> values = new ArrayList<>();
        for (String header : headers) {
            values.add(row.getOrDefault(header, ""));
        }
        return values;
    }
}

CsvController.java (Controller Layer)
java
Copy
Edit
package com.example.csvmerger.controller;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;

@RestController
@RequestMapping("/api/csv")
public class CsvController {

    @Autowired
    private CsvMergeService csvMergeService;

    @PostMapping("/merge")
    public ResponseEntity<Resource> mergeCsvFiles(@RequestParam("initial") MultipartFile initialFile,
                                                  @RequestParam("kondor") MultipartFile kondorFile) throws Exception {
        File mergedFile = csvMergeService.mergeCsvFiles(initialFile, kondorFile);

        Resource resource = new FileSystemResource(mergedFile);
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=merged_output.csv");

        return ResponseEntity.ok()
                .headers(headers)
                .contentLength(mergedFile.length())
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(resource);
    }
}
✅ 4. CsvMergerApplication.java (Main Class)
java
Copy
Edit
package com.example.csvmerger;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class CsvMergerApplication implements CommandLineRunner {

    @Autowired
    private CsvMergeService csvMergeService;

    public static void main(String[] args) {
        SpringApplication.run(CsvMergerApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        // You can comment this if you're only using controller-based upload
        String initialPath = "C:/Users/Monisha/Desktop/initial_margin.csv";
        String kondorPath = "C:/Users/Monisha/Desktop/kondor.csv";
        String outputPath = "C:/Users/Monisha/Desktop/merged_output.csv";

        csvMergeService.mergeCsvFilesFromPaths(initialPath, kondorPath, outputPath);
        System.out.println("✅ File merged and saved to: " + outputPath);
    }
}


mvn dependency:get -Dartifact=org.apache.commons:commons-csv:1.10.0



package com.example.csvmerger.service;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;

@Service
public class CsvMergeService {

    public File mergeCsvFiles(MultipartFile initialMarginFile, MultipartFile kondorFile) throws IOException {
        List<Map<String, String>> mergedRecords = new ArrayList<>();

        // Read initial margin file
        Reader reader1 = new InputStreamReader(initialMarginFile.getInputStream(), StandardCharsets.UTF_8);
        Iterable<CSVRecord> initialMarginRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader1);
        List<String> initialHeaders = new ArrayList<>(initialMarginRecords.iterator().next().toMap().keySet());

        // Read kondor file
        Reader reader2 = new InputStreamReader(kondorFile.getInputStream(), StandardCharsets.UTF_8);
        Iterable<CSVRecord> kondorRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader2);
        List<String> kondorHeaders = new ArrayList<>(kondorRecords.iterator().next().toMap().keySet());

        // Add all rows from initial margin
        for (CSVRecord record : initialMarginRecords) {
            Map<String, String> row = new LinkedHashMap<>();
            row.put("type nature", "OP");
            row.put("site code", "3428");
            row.put("application code", record.get("application code"));
            row.put("instrument code", record.get("instrument code"));
            row.put("base currency", record.get("base currency"));
            row.put("call amount", record.get("call amount"));
            row.put("rate", record.get("rate"));

            // Add the rest
            for (String header : initialHeaders) {
                if (!row.containsKey(header)) {
                    row.put(header, record.get(header));
                }
            }
            mergedRecords.add(row);
        }

        // Add all rows from kondor
        for (CSVRecord record : kondorRecords) {
            Map<String, String> row = new LinkedHashMap<>();
            row.put("type nature", record.get("type nature"));
            row.put("site code", record.get("site code"));
            row.put("application code", record.get("application code"));
            row.put("instrument code", record.get("instrument code"));
            row.put("base currency", record.get(22));
            row.put("call amount", record.get(21));
            row.put("rate", record.get(80));

            for (String header : kondorHeaders) {
                if (!row.containsKey(header)) {
                    row.put(header, record.get(header));
                }
            }
            mergedRecords.add(row);
        }

        // Final header list
        LinkedHashSet<String> headers = new LinkedHashSet<>();
        headers.add("type nature");
        headers.add("site code");
        headers.add("application code");
        headers.add("instrument code");
        headers.add("base currency");
        headers.add("call amount");
        headers.add("rate");

        for (Map<String, String> record : mergedRecords) {
            headers.addAll(record.keySet());
        }

        // Write to file
        File outputFile = File.createTempFile("merged", ".csv");
        BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile));
        CSVPrinter printer = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])));

        for (Map<String, String> record : mergedRecords) {
            List<String> row = new ArrayList<>();
            for (String header : headers) {
                row.add(record.getOrDefault(header, ""));
            }
            printer.printRecord(row);
        }
        printer.flush();
        printer.close();

        return outputFile;
    }
}



public void mergeCsvFilesFromPaths(String initialPath, String kondorPath, String outputPath) throws IOException {
    File initialFile = new File(initialPath);
    File kondorFile = new File(kondorPath);

    try (
        InputStream initialStream = new FileInputStream(initialFile);
        InputStream kondorStream = new FileInputStream(kondorFile)
    ) {
        // Wrap files as MultipartFile using a helper (Spring does not do this by default)
        MultipartFile initialMultipart = new MockMultipartFile("initial", initialFile.getName(), "text/csv", initialStream);
        MultipartFile kondorMultipart = new MockMultipartFile("kondor", kondorFile.getName(), "text/csv", kondorStream);

        File merged = mergeCsvFiles(initialMultipart, kondorMultipart);

        // Save merged file to desired output path
        try (InputStream in = new FileInputStream(merged);
             OutputStream out = new FileOutputStream(outputPath)) {
            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = in.read(buffer)) != -1) {
                out.write(buffer, 0, bytesRead);
            }
        }
    }
}





package com.example.csvmerger.service;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.mock.web.MockMultipartFile;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.StreamSupport;

@Service
public class CsvMergeService {

    public File mergeCsvFiles(MultipartFile initialMarginFile, MultipartFile kondorFile) throws IOException {
        List<Map<String, String>> mergedRecords = new ArrayList<>();

        // CSV format with header normalization
        CSVFormat csvFormat = CSVFormat.DEFAULT
                .withFirstRecordAsHeader()
                .withIgnoreHeaderCase()
                .withTrim();

        // --- Read initial margin file ---
        Reader reader1 = new InputStreamReader(initialMarginFile.getInputStream(), StandardCharsets.UTF_8);
        Iterable<CSVRecord> initialMarginIterable = csvFormat.parse(reader1);
        List<CSVRecord> initialMarginRecords = StreamSupport.stream(initialMarginIterable.spliterator(), false)
                .collect(Collectors.toList());

        if (initialMarginRecords.isEmpty()) {
            throw new IllegalArgumentException("Initial margin CSV is empty.");
        }

        Set<String> initialHeaders = initialMarginRecords.get(0).toMap().keySet();
        System.out.println("Initial Margin Headers: " + initialHeaders);

        for (CSVRecord record : initialMarginRecords) {
            Map<String, String> row = new LinkedHashMap<>();
            row.put("type nature", "OP");
            row.put("site code", "3428");

            if (!record.isMapped("application code") || !record.isMapped("instrument code") ||
                !record.isMapped("base currency") || !record.isMapped("call amount") || !record.isMapped("rate")) {
                throw new IllegalArgumentException("Missing required headers in initial margin file.");
            }

            row.put("application code", record.get("application code"));
            row.put("instrument code", record.get("instrument code"));
            row.put("base currency", record.get("base currency"));
            row.put("call amount", record.get("call amount"));
            row.put("rate", record.get("rate"));

            // Add other columns
            for (String header : initialHeaders) {
                if (!row.containsKey(header)) {
                    row.put(header, record.get(header));
                }
            }
            mergedRecords.add(row);
        }

        // --- Read kondor file ---
        Reader reader2 = new InputStreamReader(kondorFile.getInputStream(), StandardCharsets.UTF_8);
        Iterable<CSVRecord> kondorIterable = csvFormat.parse(reader2);
        List<CSVRecord> kondorRecords = StreamSupport.stream(kondorIterable.spliterator(), false)
                .collect(Collectors.toList());

        if (kondorRecords.isEmpty()) {
            throw new IllegalArgumentException("Kondor CSV is empty.");
        }

        Set<String> kondorHeaders = kondorRecords.get(0).toMap().keySet();
        System.out.println("Kondor Headers: " + kondorHeaders);

        for (CSVRecord record : kondorRecords) {
            Map<String, String> row = new LinkedHashMap<>();

            // Defensive checks to avoid IndexOutOfBoundsException
            row.put("type nature", record.isMapped("type nature") ? record.get("type nature") : "");
            row.put("site code", record.isMapped("site code") ? record.get("site code") : "");
            row.put("application code", record.isMapped("application code") ? record.get("application code") : "");
            row.put("instrument code", record.isMapped("instrument code") ? record.get("instrument code") : "");

            // Using index fallback only if absolutely needed (validate file format beforehand)
            row.put("base currency", record.size() > 22 ? record.get(22) : "");
            row.put("call amount", record.size() > 21 ? record.get(21) : "");
            row.put("rate", record.size() > 80 ? record.get(80) : "");

            for (String header : kondorHeaders) {
                if (!row.containsKey(header)) {
                    row.put(header, record.get(header));
                }
            }
            mergedRecords.add(row);
        }

        // Final header list
        LinkedHashSet<String> headers = new LinkedHashSet<>();
        headers.add("type nature");
        headers.add("site code");
        headers.add("application code");
        headers.add("instrument code");
        headers.add("base currency");
        headers.add("call amount");
        headers.add("rate");

        for (Map<String, String> record : mergedRecords) {
            headers.addAll(record.keySet());
        }

        // Write to output file
        File outputFile = File.createTempFile("merged", ".csv");
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile));
             CSVPrinter printer = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])))) {

            for (Map<String, String> record : mergedRecords) {
                List<String> row = headers.stream()
                        .map(h -> record.getOrDefault(h, ""))
                        .collect(Collectors.toList());
                printer.printRecord(row);
            }
        }

        return outputFile;
    }

    public void mergeCsvFilesFromPaths(String initialPath, String kondorPath, String outputPath) throws IOException {
        File initialFile = new File(initialPath);
        File kondorFile = new File(kondorPath);

        try (
            InputStream initialStream = new FileInputStream(initialFile);
            InputStream kondorStream = new FileInputStream(kondorFile)
        ) {
            MultipartFile initialMultipart = new MockMultipartFile("initial", initialFile.getName(), "text/csv", initialStream);
            MultipartFile kondorMultipart = new MockMultipartFile("kondor", kondorFile.getName(), "text/csv", kondorStream);

            File merged = mergeCsvFiles(initialMultipart, kondorMultipart);

            try (InputStream in = new FileInputStream(merged);
                 OutputStream out = new FileOutputStream(outputPath)) {
                byte[] buffer = new byte[1024];
                int bytesRead;
                while ((bytesRead = in.read(buffer)) != -1) {
                    out.write(buffer, 0, bytesRead);
                }
            }
        }
    }
}


recent 
package com.example.csvmerger.service;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.mock.web.MockMultipartFile;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;

@Service
public class CsvMergeService {

    public File mergeCsvFiles(MultipartFile initialMarginFile, MultipartFile kondorFile) throws IOException {
        List<Map<String, String>> mergedRecords = new ArrayList<>();

        // Read initial margin file
        Reader reader1 = new InputStreamReader(initialMarginFile.getInputStream(), StandardCharsets.UTF_8);
        Iterable<CSVRecord> initialMarginRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader1);
        Iterator<CSVRecord> initialIterator = initialMarginRecords.iterator();
        if (!initialIterator.hasNext()) {
            throw new IllegalArgumentException("Initial margin file is empty or invalid.");
        }
        List<String> initialHeaders = new ArrayList<>(initialIterator.next().toMap().keySet());

        // Read kondor file
        Reader reader2 = new InputStreamReader(kondorFile.getInputStream(), StandardCharsets.UTF_8);
        Iterable<CSVRecord> kondorRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader2);
        Iterator<CSVRecord> kondorIterator = kondorRecords.iterator();
        if (!kondorIterator.hasNext()) {
            throw new IllegalArgumentException("Kondor file is empty or invalid.");
        }
        List<String> kondorHeaders = new ArrayList<>(kondorIterator.next().toMap().keySet());

        // Re-read the initialMargin file since we moved the iterator once
        reader1 = new InputStreamReader(initialMarginFile.getInputStream(), StandardCharsets.UTF_8);
        initialMarginRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader1);

        // Process initial margin file
        for (CSVRecord record : initialMarginRecords) {
            Map<String, String> row = new LinkedHashMap<>();
            row.put("type nature", "OP");
            row.put("site code", "3428");
            row.put("application code", "3428");

            String instrumentCode = "NULL";
            try {
                instrumentCode = record.get("instrument code");
            } catch (IllegalArgumentException ignored) {}
            row.put("instrument code", instrumentCode);

            row.put("base currency", record.get("base currency"));
            row.put("call amount", record.get("call amount"));
            row.put("rate", record.get("rate"));

            // Include other headers if any
            for (String header : initialHeaders) {
                if (!row.containsKey(header)) {
                    row.put(header, record.get(header));
                }
            }

            mergedRecords.add(row);
        }

        // Process kondor file
        for (CSVRecord record : kondorRecords) {
            Map<String, String> row = new LinkedHashMap<>();
            row.put("type nature", record.get("type nature"));
            row.put("site code", record.get("site code"));
            row.put("application code", record.get("application code"));
            row.put("instrument code", record.get("instrument code"));
            row.put("base currency", record.get(22));
            row.put("call amount", record.get(21));
            row.put("rate", record.get(80));

            for (String header : kondorHeaders) {
                if (!row.containsKey(header)) {
                    row.put(header, record.get(header));
                }
            }

            mergedRecords.add(row);
        }

        // Build final header list
        LinkedHashSet<String> headers = new LinkedHashSet<>();
        headers.add("type nature");
        headers.add("site code");
        headers.add("application code");
        headers.add("instrument code");
        headers.add("base currency");
        headers.add("call amount");
        headers.add("rate");

        for (Map<String, String> record : mergedRecords) {
            headers.addAll(record.keySet());
        }

        // Write to CSV
        File outputFile = File.createTempFile("merged", ".csv");
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile));
             CSVPrinter printer = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])))) {

            for (Map<String, String> record : mergedRecords) {
                List<String> row = new ArrayList<>();
                for (String header : headers) {
                    row.add(record.getOrDefault(header, ""));
                }
                printer.printRecord(row);
            }
        }

        return outputFile;
    }

    public void mergeCsvFilesFromPaths(String initialPath, String kondorPath, String outputPath) throws IOException {
        File initialFile = new File(initialPath);
        File kondorFile = new File(kondorPath);

        try (
            InputStream initialStream = new FileInputStream(initialFile);
            InputStream kondorStream = new FileInputStream(kondorFile)
        ) {
            MultipartFile initialMultipart = new MockMultipartFile("initial", initialFile.getName(), "text/csv", initialStream);
            MultipartFile kondorMultipart = new MockMultipartFile("kondor", kondorFile.getName(), "text/csv", kondorStream);

            File merged = mergeCsvFiles(initialMultipart, kondorMultipart);

            try (InputStream in = new FileInputStream(merged);
                 OutputStream out = new FileOutputStream(outputPath)) {
                byte[] buffer = new byte[1024];
                int bytesRead;
                while ((bytesRead = in.read(buffer)) != -1) {
                    out.write(buffer, 0, bytesRead);
                }
            }
        }
    }
}


// Required headers in initial margin file
List<String> requiredHeaders = Arrays.asList("base currency", "call amount", "rate");

Set<String> initialHeaderSet = new HashSet<>(initialHeaders);
for (String required : requiredHeaders) {
    if (!initialHeaderSet.contains(required)) {
        throw new IllegalArgumentException("Missing required header in initial margin file: " + required);
    }
}


package com.example.csvmerger.service;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.ArrayList;
import java.util.List;

@Service
public class CsvMergeService {

    public void mergeCsvFiles(Path initialMarginPath, Path kondorPath, Path outputPath) {
        List<String[]> mergedRows = new ArrayList<>();
        List<String> headers = List.of("Application Code", "Instrument Code", "Base Currency", "Call Amount", "Rate", "Type Nature");

        try (
                Reader initialReader = Files.newBufferedReader(initialMarginPath);
                Reader kondorReader = Files.newBufferedReader(kondorPath);
                Writer writer = Files.newBufferedWriter(outputPath);
                CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])))
        ) {
            // Read Initial Margin
            Iterable<CSVRecord> initialRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(initialReader);
            for (CSVRecord record : initialRecords) {
                String applicationCode = "3428";
                String instrumentCode = "NULL"; // Not in initial margin
                String baseCurrency = record.get("Base Currency").trim();
                String callAmount = record.get("Call Amount").trim();
                String rate = record.get("Rate").trim();
                String typeNature = "OP"; // Not in initial margin

                mergedRows.add(new String[]{applicationCode, instrumentCode, baseCurrency, callAmount, rate, typeNature});
            }

            // Read Kondor
            Iterable<CSVRecord> kondorRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(kondorReader);
            for (CSVRecord record : kondorRecords) {
                String applicationCode = "3428";
                String instrumentCode = record.isMapped("Instrument Code") ? record.get("Instrument Code").trim() : "NULL";
                String baseCurrency = record.size() > 22 ? record.get(22).trim() : ""; // Column 22 index for Base Currency
                String callAmount = ""; // Not in kondor
                String rate = "";       // Not in kondor
                String typeNature = record.isMapped("Type Nature") ? record.get("Type Nature").trim() : "NULL";

                mergedRows.add(new String[]{applicationCode, instrumentCode, baseCurrency, callAmount, rate, typeNature});
            }

            // Write to merged CSV
            for (String[] row : mergedRows) {
                csvPrinter.printRecord((Object[]) row);
            }

            csvPrinter.flush();
        } catch (Exception e) {
            throw new IllegalArgumentException("Error while merging CSV files: " + e.getMessage(), e);
        }
    }
}


package com.example.csvmerger.service;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.commons.csv.CSVPrinter;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.*;

@Service
public class CsvMergeService {

    public void mergeCsvFiles(Path initialMarginPath, Path kondorPath, Path outputPath) {
        try (
                Reader reader1 = Files.newBufferedReader(initialMarginPath);
                Reader reader2 = Files.newBufferedReader(kondorPath);
                CSVParser parser1 = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader1);
                CSVParser parser2 = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(reader2);
                Writer writer = Files.newBufferedWriter(outputPath);
                CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(getMergedHeaders(parser1, parser2)))
        ) {
            List<String> headers = getMergedHeaders(parser1, parser2);

            for (CSVRecord record : parser1) {
                Map<String, String> row = new LinkedHashMap<>();
                for (String header : headers) {
                    String value;
                    if (header.equalsIgnoreCase("Application Code")) {
                        value = "3428";
                    } else if (header.equalsIgnoreCase("Instrument Code")) {
                        value = ""; // NULL in CSV is usually left blank
                    } else if (header.equalsIgnoreCase("Type Nature")) {
                        value = "OP";
                    } else {
                        value = getIgnoreCase(record, header);
                    }
                    row.put(header, value);
                }
                csvPrinter.printRecord(row.values());
            }

            for (CSVRecord record : parser2) {
                Map<String, String> row = new LinkedHashMap<>();
                for (String header : headers) {
                    String value;
                    if (header.equalsIgnoreCase("Application Code")) {
                        value = ""; // Not present in Kondor
                    } else if (header.equalsIgnoreCase("Instrument Code")) {
                        value = getIgnoreCase(record, header);
                    } else if (header.equalsIgnoreCase("Type Nature")) {
                        value = getIgnoreCase(record, header);
                    } else {
                        value = getIgnoreCase(record, header);
                    }
                    row.put(header, value);
                }
                csvPrinter.printRecord(row.values());
            }

        } catch (IOException e) {
            throw new RuntimeException("Error merging CSV files: " + e.getMessage(), e);
        }
    }

    private List<String> getMergedHeaders(CSVParser parser1, CSVParser parser2) {
        Set<String> headerSet = new LinkedHashSet<>();

        parser1.getHeaderMap().keySet().forEach(h -> headerSet.add(capitalizeFully(h)));
        parser2.getHeaderMap().keySet().forEach(h -> headerSet.add(capitalizeFully(h)));

        headerSet.add("Application Code");
        headerSet.add("Instrument Code");
        headerSet.add("Type Nature");

        return new ArrayList<>(headerSet);
    }

    private String getIgnoreCase(CSVRecord record, String header) {
        for (String h : record.toMap().keySet()) {
            if (h.equalsIgnoreCase(header)) {
                return record.get(h);
            }
        }
        return "";
    }

    private String capitalizeFully(String input) {
        String[] parts = input.toLowerCase().split(" ");
        StringBuilder sb = new StringBuilder();
        for (String part : parts) {
            if (!part.isEmpty()) {
                sb.append(Character.toUpperCase(part.charAt(0))).append(part.substring(1)).append(" ");
            }
        }
        return sb.toString().trim();
    }
}



// CsvMergerApplication.java
package com.example.csvmerger;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.nio.file.Path;

@SpringBootApplication
public class CsvMergerApplication implements CommandLineRunner {

    @Autowired
    private CsvMergeService csvMergeService;

    public static void main(String[] args) {
        SpringApplication.run(CsvMergerApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        // Optional: You can disable this block if you're only using the controller for uploads
        String initialPath = "C:/Users/h59606/Downloads/Initial_Margin_E0D_20250307.csv";
        String kondorPath = "C:/Users/h59606/Downloads/KONDORFX.csv";
        String outputPath = "C:/Users/h59606/Downloads/merged_output.csv";

        csvMergeService.mergeCsvFiles(Path.of(initialPath), Path.of(kondorPath), Path.of(outputPath));

        System.out.println("File merged and saved to: " + outputPath);
    }
}


// CsvMergeService.java
package com.example.csvmerger.service;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.util.*;

@Service
public class CsvMergeService {
    public ByteArrayInputStream mergeCsvFiles(MultipartFile initialMargin, MultipartFile kondor) throws IOException {
        Reader initialReader = new InputStreamReader(initialMargin.getInputStream());
        Reader kondorReader = new InputStreamReader(kondor.getInputStream());

        Iterable<CSVRecord> initialRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(initialReader);
        Iterable<CSVRecord> kondorRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(kondorReader);

        Set<String> headers = new LinkedHashSet<>();
        headers.addAll(Arrays.asList("Application Code", "Type Nature"));

        for (CSVRecord record : initialRecords) {
            headers.addAll(record.toMap().keySet());
        }
        for (CSVRecord record : kondorRecords) {
            headers.addAll(record.toMap().keySet());
        }

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        CSVPrinter printer = new CSVPrinter(new PrintWriter(out), CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])));

        // Reset Readers for second pass
        initialReader = new InputStreamReader(initialMargin.getInputStream());
        kondorReader = new InputStreamReader(kondor.getInputStream());
        initialRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(initialReader);
        kondorRecords = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(kondorReader);

        for (CSVRecord record : initialRecords) {
            Map<String, String> map = new LinkedHashMap<>();
            map.put("Application Code", "3428");
            map.put("Type Nature", "OP");
            for (String h : headers) {
                if (!map.containsKey(h)) {
                    map.put(h, record.toMap().getOrDefault(h, ""));
                }
            }
            printer.printRecord(map.values());
        }

        for (CSVRecord record : kondorRecords) {
            Map<String, String> map = new LinkedHashMap<>();
            map.put("Application Code", "");
            map.put("Type Nature", record.toMap().getOrDefault("Type Nature", ""));
            for (String h : headers) {
                if (!map.containsKey(h)) {
                    map.put(h, record.toMap().getOrDefault(h, ""));
                }
            }
            printer.printRecord(map.values());
        }

        printer.flush();
        return new ByteArrayInputStream(out.toByteArray());
    }
}


// CsvController.java
package com.example.csvmerger.controller;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayInputStream;

@RestController
@RequestMapping("/api/csv")
public class CsvController {

    @Autowired
    private CsvMergeService csvMergeService;

    @PostMapping("/merge")
    public ResponseEntity<byte[]> mergeCsv(
            @RequestParam("initial") MultipartFile initial,
            @RequestParam("kondor") MultipartFile kondor) throws Exception {

        ByteArrayInputStream stream = csvMergeService.mergeCsvFiles(initial, kondor);
        byte[] output = stream.readAllBytes();

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=merged.csv")
                .contentType(MediaType.parseMediaType("application/csv"))
                .body(output);
    }
}


I've provided the full, updated code for your Spring Boot application, including:

1. CsvMergerApplication.java – Main entry point.


2. CsvMergeService.java – Handles the merging logic.


3. CsvController.java – REST controller that accepts two CSVs and returns the merged file.



Let me know if you'd like help testing it or creating a minimal pom.xml to ensure dependencies like commons-csv are set up correctly.


package com.example.csvmerger.service;

import org.apache.commons.csv.*;
import org.springframework.stereotype.Service;

import java.io.*;
import java.nio.file.Path;
import java.util.*;

@Service
public class CsvMergeService {

    public void mergeCsvFiles(Path initialPath, Path kondorPath, Path outputPath) throws IOException {

        List<CSVRecord> initialRecords;
        List<CSVRecord> kondorRecords;

        try (
            Reader initialReader = new FileReader(initialPath.toFile());
            Reader kondorReader = new FileReader(kondorPath.toFile())
        ) {
            CSVParser initialParser = CSVFormat.DEFAULT
                    .withFirstRecordAsHeader()
                    .withTrim()
                    .parse(initialReader);

            CSVParser kondorParser = CSVFormat.DEFAULT
                    .withFirstRecordAsHeader()
                    .withTrim()
                    .parse(kondorReader);

            initialRecords = initialParser.getRecords();
            kondorRecords = kondorParser.getRecords();

            Set<String> headers = new LinkedHashSet<>();
            headers.addAll(initialParser.getHeaderMap().keySet());
            headers.addAll(kondorParser.getHeaderMap().keySet());

            headers.add("Application Code");
            headers.add("Type Nature");

            // Normalize key headers
            String baseCurrencyHeader = getMatchingHeader(headers, "Base Currency");
            String callAmountHeader = getMatchingHeader(headers, "Call Amount");
            String rateHeader = getMatchingHeader(headers, "Rate");
            String instrumentCodeHeader = getMatchingHeader(headers, "Instrument Code");

            try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputPath.toFile()));
                 CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])))) {

                for (CSVRecord record : initialRecords) {
                    Map<String, String> row = new LinkedHashMap<>();
                    for (String header : headers) {
                        String value = record.isMapped(header) ? record.get(header) : "";
                        row.put(header, value);
                    }

                    row.put("Application Code", "3428");
                    row.put("Type Nature", "OP");

                    // Fill instrument code if not present
                    if (!headers.contains(instrumentCodeHeader) || row.get(instrumentCodeHeader).isEmpty()) {
                        row.put(instrumentCodeHeader, "NULL");
                    }

                    csvPrinter.printRecord(row.values());
                }

                for (CSVRecord record : kondorRecords) {
                    Map<String, String> row = new LinkedHashMap<>();
                    for (String header : headers) {
                        String value = record.isMapped(header) ? record.get(header) : "";
                        row.put(header, value);
                    }

                    // Application Code is only for initial
                    row.put("Application Code", "");
                    // Get actual Type Nature from Kondor
                    row.put("Type Nature", record.isMapped("Type Nature") ? record.get("Type Nature") : "");

                    csvPrinter.printRecord(row.values());
                }
            }
        }
    }

    private String getMatchingHeader(Set<String> headers, String key) {
        for (String h : headers) {
            if (h.equalsIgnoreCase(key)) {
                return h;
            }
        }
        return key; // fallback
    }
}


package com.example.csvmerger.controller;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;

@RestController
@RequestMapping("/api/csv")
public class CsvController {

    @Autowired
    private CsvMergeService csvMergeService;

    @PostMapping(value = "/merge", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<byte[]> mergeCsvFiles(
            @RequestParam("initialFile") MultipartFile initialFile,
            @RequestParam("kondorFile") MultipartFile kondorFile) throws IOException {

        // Save both files to temporary locations
        Path tempInitial = Files.createTempFile("initial-", ".csv");
        Path tempKondor = Files.createTempFile("kondor-", ".csv");
        Path tempOutput = Files.createTempFile("merged-", ".csv");

        initialFile.transferTo(tempInitial.toFile());
        kondorFile.transferTo(tempKondor.toFile());

        // Merge logic
        csvMergeService.mergeCsvFiles(tempInitial, tempKondor, tempOutput);

        byte[] mergedBytes = Files.readAllBytes(tempOutput);

        // Clean up temporary files
        Files.deleteIfExists(tempInitial);
        Files.deleteIfExists(tempKondor);
        Files.deleteIfExists(tempOutput);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"merged.csv\"")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(mergedBytes);
    }
}


updated
package com.example.csvmerger.service;

import org.apache.commons.csv.*;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.*;

@Service
public class CsvMergeService {

    public ByteArrayOutputStream mergeCsvFiles(MultipartFile initialFile, MultipartFile kondorFile) throws IOException {
        try (
                Reader initialReader = new InputStreamReader(initialFile.getInputStream());
                Reader kondorReader = new InputStreamReader(kondorFile.getInputStream());
                ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
                Writer writer = new OutputStreamWriter(outputStream)
        ) {
            CSVParser initialParser = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(initialReader);
            CSVParser kondorParser = CSVFormat.DEFAULT.withFirstRecordAsHeader().parse(kondorReader);

            // Headers
            Set<String> headers = new LinkedHashSet<>();
            headers.add("Site Code");
            headers.add("Application Code");
            headers.add("Type Nature");
            headers.add("Base Currency");

            headers.addAll(initialParser.getHeaderMap().keySet());
            headers.addAll(kondorParser.getHeaderMap().keySet());

            CSVPrinter printer = new CSVPrinter(writer, CSVFormat.DEFAULT.withHeader(headers.toArray(new String[0])));

            // Initial File Rows
            for (CSVRecord record : initialParser) {
                Map<String, String> row = new HashMap<>();

                for (String header : initialParser.getHeaderMap().keySet()) {
                    row.put(header, record.get(header));
                }

                row.put("Site Code", "3428");
                row.put("Application Code", ""); // empty for initial
                row.put("Type Nature", "OP");

                // Add Base Currency from its own column
                row.put("Base Currency", record.get("Base Currency"));

                List<String> orderedRow = new ArrayList<>();
                for (String header : headers) {
                    orderedRow.add(row.getOrDefault(header, ""));
                }

                printer.printRecord(orderedRow);
            }

            // Kondor File Rows
            for (CSVRecord record : kondorParser) {
                Map<String, String> row = new HashMap<>();

                for (String header : kondorParser.getHeaderMap().keySet()) {
                    row.put(header, record.get(header));
                }

                row.put("Site Code", record.get("Site Code"));
                row.put("Application Code", record.get("Application Code"));
                row.put("Type Nature", record.get("Type Nature"));

                // Base Currency is in column 22 (index 21)
                String baseCurrency = record.size() > 21 ? record.get(21) : "";
                row.put("Base Currency", baseCurrency);

                List<String> orderedRow = new ArrayList<>();
                for (String header : headers) {
                    orderedRow.add(row.getOrDefault(header, ""));
                }

                printer.printRecord(orderedRow);
            }

            printer.flush();
            return outputStream;
        }
    }

    // Optional CLI method for merging by path
    public void mergeCsvFiles(Path initialPath, Path kondorPath, Path outputPath) throws IOException {
        MultipartFile initialFile = new MockMultipartFile("initial", Files.readAllBytes(initialPath));
        MultipartFile kondorFile = new MockMultipartFile("kondor", Files.readAllBytes(kondorPath));
        ByteArrayOutputStream merged = mergeCsvFiles(initialFile, kondorFile);
        Files.write(outputPath, merged.toByteArray());
    }
}



1. CsvMergerApplication.java

package com.example.csvmerger;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.nio.file.Path;

@SpringBootApplication
public class CsvMergerApplication implements CommandLineRunner {

    @Autowired
    private CsvMergeService csvMergeService;

    public static void main(String[] args) {
        SpringApplication.run(CsvMergerApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        // Uncomment to test without POST API
        /*
        String initialPath = "C:\\Users\\h59606\\Downloads\\Initial_Margin_E0D_20250307.csv";
        String kondorPath = "C:\\Users\\h59606\\Downloads\\KONDORFX.csv";
        String outputPath = "C:\\Users\\h59606\\Downloads\\merged_output.csv";

        csvMergeService.mergeCsvFiles(Path.of(initialPath), Path.of(kondorPath), Path.of(outputPath));
        System.out.println("File merged and saved to: " + outputPath);
        */
    }
}


---

2. CsvController.java

package com.example.csvmerger.controller;

import com.example.csvmerger.service.CsvMergeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ByteArrayResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayOutputStream;

@RestController
@RequestMapping("/api/csv")
public class CsvController {

    @Autowired
    private CsvMergeService csvMergeService;

    @PostMapping("/merge")
    public ResponseEntity<ByteArrayResource> mergeCsvFiles(
            @RequestParam("initial") MultipartFile initialFile,
            @RequestParam("kondor") MultipartFile kondorFile) throws Exception {

        ByteArrayOutputStream mergedCsv = csvMergeService.mergeCsvFiles(initialFile, kondorFile);

        ByteArrayResource resource = new ByteArrayResource(mergedCsv.toByteArray());

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=merged_output.csv")
                .contentType(MediaType.parseMediaType("text/csv"))
                .body(resource);
    }
}


---