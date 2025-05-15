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

