package org.example.coursenavigator;

import javafx.event.ActionEvent;
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.Label;
import javafx.scene.control.TreeItem;
import javafx.scene.control.TreeView;
import javafx.scene.input.MouseEvent;
import javafx.scene.web.WebView;

import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.*;

public class MainController implements Initializable {

    public TreeItem rootItem;
    @FXML
    private WebView contentWebView;

    @FXML
    private TreeView<String> courseTree;
    @FXML
    private Label statusLabel;

    // ИЗМЕНЕНИЕ 1: Теперь это пустая HashMap вместо жесткого Map.of
    private Map<String, String> labToFileMap = new HashMap<>();

    @FXML
    private void handleTreeItemClick(MouseEvent event) {
        TreeItem<String> item = courseTree.getSelectionModel().getSelectedItem();

        if (item != null && labToFileMap.containsKey(item.getValue())) {
            String mdFile = labToFileMap.get(item.getValue());
            loadAndDisplayMarkdown(mdFile);
        }
    }

    private void loadAndDisplayMarkdown(String mdFilePath) {
        statusLabel.setText("Загрузка " + mdFilePath + "...");

        String mdContent = null;

        try {
            mdContent = loadFromLocalFile(mdFilePath);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        String htmlContent = MarkdownRenderer.toHtml(mdContent);
        String fullHtml = MarkdownRenderer.wrapWithHtmlTemplate(htmlContent);
        contentWebView.getEngine().loadContent(fullHtml);

        statusLabel.setText("Загружено: " + mdFilePath);
    }

    private String loadFromLocalFile(String filePath) throws IOException {
        Path path = Paths.get("", filePath);
        return Files.readString(path, StandardCharsets.UTF_8);
    }

    @FXML
    private void handleRefresh() {
        // Обновить все MD файлы из GitHub
        //CacheManager.refreshAll();
    }

    public void toggleOfflineMode(ActionEvent actionEvent) {
        //офлайн режим
    }

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle) {
        // ИЗМЕНЕНИЕ 2: Динамически загружаем файлы вместо жесткого списка
        File labsDir = new File("course-materials/labs");
        File[] mdFiles = labsDir.listFiles((dir, name) -> name.endsWith(".md"));

        if (mdFiles != null) {
            for (File mdFile : mdFiles) {
                String fileName = mdFile.getName();
                String labNumber = fileName.replaceAll("lab", "").replaceAll("\\.md", "");
                String labDisplayName = "Лаб " + labNumber + ".";
                String filePath = "course-materials/labs/" + fileName;

                labToFileMap.put(labDisplayName, filePath);

                TreeItem<String> child = new TreeItem<>(labDisplayName);
                rootItem.getChildren().add(child);
            }
        }

        courseTree.setRoot(rootItem);
    }
}
